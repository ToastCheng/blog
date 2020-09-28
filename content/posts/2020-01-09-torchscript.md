---
title:  "Torchscript: train in Python, run in C++"
date:   2020-01-09T13:32:56+0800
tags: [torchscript, pytorch, c++]
categories: technique
---
This article is mainly about serialize a pytorch model, and load it into C++.

### **First of all, traced your model**

"Trace" is a mechanism which pytorch used to record the behavior of neural nets. 
To perform tracing, first you have to initialize your model, say that you have trained your model until it reach production level.

```python
import torch
from your.model.path import Model

# assume that Model() is simply a network class which inherit nn.Module.
model = Model()

# of course you have to load the weight of your model.
weight = torch.load("{your-weight-path}", map_location=torch.device("cpu"))
model.load_state_dict(weight)
```

Then you have to make an `example`, i.e., a tensor that match the size of `model`'s input size. Note that you will have to set `model` to `eval()` for disabling training features since we are doing this for inference.

```python
example = torch.rand(1, 3, 512, 512)
model.eval()
```

After configuration, just simply called `torch.jit.trace` and pass the model and example to it. Once the tracing is done successfully, you can saved it into file.

```python
traced = torch.jit.trace(model, example, check_trace=True)
traced.save("traced_model.pt")
```

You can make a simple check to see if the output is the same as the original model.


```python
traced_model = torch.jit.load("traced_model.pt")

model(example)
# output: 
# tensor([[-10.5246, -15.4433, -15.8299, -14.9295, -13.4168, -10.2231]],
#        grad_fn=<AddmmBackward>)
traced_model(example)
# output: 
# tensor([[-10.5246, -15.4433, -15.8299, -14.9295, -13.4168, -10.2231]],
#        grad_fn=<AddmmBackward>)
```

*note 1*
The traced torchscript file does not seems to be platform-independent, if you trace it on Mac, and load it in Windows, unpredictable errors happens when loading model.

*note 2*
As offical document stated, if your model contains control flows(`if`, `else`), or any operations that depends on the value of `example`, i.e., `example` cannot reach some parts in your model, the traced model will not perform the same as the original model. To tackle this, you will need to add `@torch.jit.script` in every function which contains control flow.

e.g.
```python
class Model(nn.Module):
    ...
    def forward(self, x):
        ...
        x = self._some_operation(x)
        return x
    @torch.jit.script
    def _some_operation(self, x):
        if x.sum() == 0:
            ...
        else:
            ...
```

*note 3*
If your model contains custom `autograd.Function`, the operation cannot be traced (yet). One solution is to write a C++ operation and build pytorch by yourself like [this](http://lernapparat.de/pytorch-traceable-differentiable/), or you can simply try to use existed operations to build your custom operation rather than inherit `autograd.Function`.

### **Second, write C++ inference code**

Though the official document has made it pretty stright forward, I do have some hard time building it in Windows (no problem in unix-like). So I will focus on the steps that need to perform on Windows:

1. Download source:
    * cpu:
    ```
    wget https://download.pytorch.org/libtorch/nightly/cpu/libtorch-shared-with-deps-latest.zip
    ```
    * cuda 9.0: 
    ```
    wget https://download.pytorch.org/libtorch/nightly/cu90/libtorch-shared-with-deps-latest.zip
    ```
2. Preparing main.cpp, here I will extend the [official example code](https://pytorch.org/tutorials/advanced/cpp_export.html). The first part below is how c++ load a traced model, simply call `torch::jit::load()`.

```cpp
#include <torch/script.h>

#include <iostream>
#include <string.h>

int main() {
  
  std::string model_path = "traced_model.pt"

  torch::jit::script::Module module;
  try {
    // Deserialize the ScriptModule from a file using torch::jit::load().
    module = torch::jit::load(model_path);
  }
  catch (const c10::Error& e) {
    std::cerr << "error loading the model\n";
    return -1;
  }

  ...
```
Say that you have an integer pointer (1-D array, flatten from a 2 or 3-D array) `int* data`, which is a way that C++ store image data. You can simply use `torch::from_blob()` to read the data into tensor, the sizes of it will be given in the second argument. The `options` is the way that `torch` used (as the third argument) to decide the type that will be read. You can find more in [here](https://pytorch.org/cppdocs/notes/tensor_creation.html#configuring-properties-of-the-tensor).

```cpp
  ... (still in main())
  
  int height = 512;
  int width = 512;
  int* data = get_image_data(); // some api that you get your data.
  
  at::Tensor t;
  auto options = c10::TensorOptions().dtype(torch::kShort);
  t = torch::from_blob(data, { 
    /*batch=*/1, 
    /*channel=*/1, 
    /*width=*/width, 
    /*height=*/height 
  }, options);
  
  ...
```

Lastly, you will have to initialize a vector of `torch::jit::IValue` as an input for a torchscript model. simply `push_back` a batch of tensor inside and you can run with `module.forward()`. You can get the result using `toTensor()`. There are some useful member functions of `Tensor`:
- observe the shape by `sizes()`
- cast the value into primitive type by `item<type>()`

```cpp
  ... (still in main())
  
  std::vector<torch::jit::IValue> input;
  input.push_back(data)
  output = module.forward(input).toTensor();
  
  // say that we want to know the output of the first input(0), second dimension(1)
  std::cout << "result: " << output[0][1].item<float>() << std::endl;
  
  // observe the shape
  std::cout << output.sizes() << std::endl;
  
  return 0;
}
```

### **Finally, build the app**

1. Preparing cmakefile
    We will have to write a CMakeLists to build the app. If you have not seen any `CMakeLists.txt` before it would be a little timidating, but it can be dissect:
- `project`
    The name of the app, usually the name of root directory.
- `set` 
    Set any argument that is needed.
- `find_package`
    Extremely handy if a third-party package has cmake support. cmake will find files like `{uppercase package name}Config.cmake` or `{lowercase package name}-config.cmake`. And You will not need to configure any lib path, include path by yourself. But you need to set `CMAKE_PREFIX_PATH` let cmake to know where to find those files. If you have multiple package you can set it like:  `CMAKE_PREFIX_PATH={path 1}:{path 2}`
- `add_executable`
    Like `g++ -o test-torch main.cpp`.
- The `MSVC` part
    For Windows OS only, provided by document.

```cmake
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(test-torch)

# if not set here, you should set it in argument with -D flag. e.g. -DCMAKE_PREFIX_PATH={libtorch path}
set(CMAKE_PREFIX_PATH "{libtorch path}")
# if not set, annoying warnings will pop up
cmake_policy(SET CMP0054 NEW)

find_package(Torch REQUIRED)

add_executable(test-torch main.cpp)
target_link_libraries(test-torch "${TORCH_LIBRARIES}")
set_property(TARGET test-torch PROPERTY CXX_STANDARD 14)

if (MSVC)
  file(GLOB TORCH_DLLS "${TORCH_INSTALL_PREFIX}/lib/*.dll")
  add_custom_command(TARGET test-torch
                     POST_BUILD
                     COMMAND ${CMAKE_COMMAND} -E copy_if_different
                     ${TORCH_DLLS}
                     $<TARGET_FILE_DIR:test-torch>)
endif (MSVC)
```

2. Build

```bash
mkdir build
cd build
# set CMAKE_PREFIX_PATH to let cmake knows where to find the cmake file of libtorch.
cmake -DCMAKE_PREFIX_PATH="{libtorch path}" ..
# OR if you set inside CMakeLists.txt you can alternatively run:
cmake ..

# build, equivalent to "make"
cmake --build . --config Release

# after building, dlls will be generated at the project root, 
# copy them to the path in where the executable is.
cp c:\...\test-torch\*.dll C:\...\test-torch\build\Release\
cd Release

# run!
.\test-torch.exe
```




reference
- [build libtorch with cmake](https://discuss.pytorch.org/t/error-running-libtorch-example-program/53980/6)
- [custom op](http://lernapparat.de/pytorch-traceable-differentiable/)
- [official c++ tutorial](https://pytorch.org/tutorials/advanced/cpp_export.html)
- [tensorOption](https://pytorch.org/cppdocs/notes/tensor_creation.html#configuring-properties-of-the-tensor)
