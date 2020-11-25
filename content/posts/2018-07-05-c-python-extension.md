
---
title:  "Note: Python C Extension"
date:   2018-07-05T16:26:09+0800
tags: [note, c++, python, extension]
categories: technique
---

Python is very handy for prototyping things and thanks to 
Thanks to dynamic typing, Python is handy for coding in an interactively way, fast prototyping projects, etc. However, Python is not efficient enough, if some part of your code is compute-intensive that needs to run efficiently like C, you should consider writing that part in C.

### Python's C extension


1. Write C source code with PyObject
  - main algorithm
  - PyMethodDef
  - PyMODINIT_FUNC
2. Write setup.py
3. Run “python setup.py build_ext --inplace”
4. Call your C function as oridinary python method!


### C -- Function

#### PyArg_ParseTuple(PyObject *args, const char *format, ...)
- args: input from python interface
- format: datatype of each argument in “args”
- …: the pointers of C variables that will be assigned value from “args”

#### Py_BuildValue(const char *format, ...)
- format: datatype of the output
- …: value that need to return

### C -- Method table
```cpp
static PyMethodDef module_methods[] = {
    {"_dtw", dtw, METH_VARARGS, "dynamic time warping"},
    {NULL, NULL, 0, NULL}
};
```

### C -- Initialization table
```cpp
PyMODINIT_FUNC PyInit_<module>(void) 
{
PyObject *module;
static struct PyModuleDef moduledef = {
        	PyModuleDef_HEAD_INIT,	// just add
        	"sequence_utils",		// module name
        	"module_docstring",		// module document
        	-1,				// global state
        	module_methods,		// method table which initialized before
        	NULL,
        	NULL,
        	NULL,
        	NULL
    	};
    module = PyModule_Create(&moduledef);
    if (!module) return NULL;

    return module;
}
```

### Python -- `setup.py`
```python
from distutils.core import setup, Extension

setup(name='dtw',
	  version='1.0',
	  description='test',
	  ext_modules=[Extension('sequence_utils', ['_dtw.cpp'])]
	  )
```