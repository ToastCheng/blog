---
title:  "Torchscript: Torch frontend note"
date:   2020-01-12T22:03:34+0800
tags: [note, pytorch, c++, machine learning]
categories: technique
---

# Torch C++ Frontend Note

First of all, include `libtorch` by
```cpp
#include <torch/script.h>
```

### Frequently used operations

- Check if cuda is available
```cpp
torch::Device device(torch::cuda::is_available() ? torch::kCUDA : torch::kCPU);
```

- Specific device (by id)
```cpp
// device 0
torch::Device device(torch::kCUDA, 0);
```


- Convert a image with (1D-pointer type) into tensor
```cpp
int width = 512;
int height = 512;
int* pixelData = (int *) malloc(width * height * sizeof(int));
some_initialization(pixelData); // do something to load the data.
at::Tensor t = torch::from_blob(pixelData, { width, height });
```

- Convert with options
```cpp
auto options = c10::TensorOptions().dtype(torch::kShort);
at::Tensor t = torch::from_blob(pixelData, { 1, width, height }, options);
```

- Type casting
```cpp
tensor = tensor.toType(torch::kFloat32);
```

- Convert tensor data into primitive types
```cpp
// get the value of tensor on index(0, 0) to float
float val = tensor[0][0].item<float>()
```

### Compilation

- CUDA
1. CUDA Toolkit
    Directly install by official installer. Once the installation is completed, the `CUDA_TOOLKIT_ROOT_DIR` will be automatically set.
2. cuDNN
    Download from official website. Set the include path `CUDNN_LIBRARY_PATH` and library path `CUDNN_INCLUDE_PATH` in `CMakeLists.txt` to where you store the cuDNN package.


### Torchscript

### Custom Dataset and DataLoader

Dataset
```cpp
class CustomDataset : public torch::data::Dataset<CustomDataset> {
  // use Batch as a alias of torch::data::Example<>
  using Batch = torch::data::Example<>;
  private:
    std::vector<std::string> file_list;

  public:
    explicit ICHDataset(const std::vector<std::string> file_list) 
      : file_list(file_list) {}

    Batch get(size_t index) {

      auto  = parse_data(index);

      return { data, label };
    } 

    torch::optional<size_t> size() const {
      return file_list.size();
    }
};
```

DataLoader

```cpp
auto dataset = ICHDataset(file_list).map(torch::data::transforms::Stack<>());;;
auto dataloader = torch::data::make_data_loader(
  std::move(dataset),
  torch::data::DataLoaderOptions()
    .batch_size(batch_size)
    .workers(2)
    .enforce_ordering(true)
  );
```





### Errors, pitfalls

```
error: ‘is_available’ is not a member of ‘at::cuda’
  if (torch::cuda::is_available())` 
```
- Include `<torch/torch.h>`, though `<torch.script.h>` might works in the main.cpp file, it does not work in other files.
    https://discuss.pytorch.org/t/torch-is-available-is-throwing-compilation-error/41158/3




- Compile CUDA version without CUDA-capable device
    https://stackoverflow.com/questions/20186848/can-i-compile-a-cuda-program-without-having-a-cuda-device/20196425