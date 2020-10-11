---
title:  "Pyarmor: Obfuscate your python source code"
date:   2020-02-11T23:13:54+0800
tags: [note, pyarmor, obfuscate, security]
categories: technique
---


[Pyarmor](https://pyarmor.readthedocs.io/en/latest/index.html) is a tool that helps you obfuscate your python source code in a jiffy.

## Usage

  - Main usage -- obfuscate
  ```bash
  pyarmor obfuscate foo.py
  ```
  This will generate `dist` with structure:
  ```bash
  dist
  ├── pytransform
  │   ├── __init__.py
  │   ├── _pytransform.dylib
  │   ├── license.lic
  │   └── pytransform.key
  └── foo.py
  ```
  The dist is simply a module that perform exact the same functionality without source code.
  By changeing the name of entry file to `__main__.py`:
  ```bash
  mv foo.py __main__.py
  ```
  And add a output flag `--output` to specify the output dir name:
  ```bash
  pyarmor obfuscate --output=foo foo.py
  ```
  The `dist` becomes:
  ```bash
  foo
  ├── pytransform
  │   ├── __init__.py
  │   ├── _pytransform.dylib
  │   ├── license.lic
  │   └── pytransform.key
  └── __main__.py
  ```
  Then you can run `python foo`, same as `python foo.py` with the source code.
  
  - Add license 
  If one use `pyarmor obfuscate foo.py` to generate code, there is a `license.lic` inside `dist/pytransform` with no limitation. But running the obfuscated code needs the present of this file. To add more limitation to the license, you can use `pyarmor licenses`:
  ```bash
  pyarmor licenses \
    --bind-disk "100304PBN2081SF3NJ5T" \
    --bind-mac "20:c1:d2:2f:a0:96" \
    --expired 2020-01-01 \
    code_name

  # then replace the origin license.
  cp licenses/code_name/license.lic dist/pytransform/
  cd dist/
  python foo.py
  ```
  However, in free trial mode, everyone who knows how to use pyarmor can generate a license himself and replace the original one.
  - Multiple platform ([doc](https://pyarmor.readthedocs.io/en/latest/advanced.html#running-obfuscated-scripts-in-multiple-platforms))
  ```bash
  pyarmor obfuscate \
    --platform windows.x86_64 \
    --platform linux.x86_64 \
    --platform darwin.x86_64 \
    foo.py
  ```
  To view the whole supported platform: [here](https://pyarmor.readthedocs.io/en/latest/platforms.html#standard-platform-names)
  - Recursively obfuscate
  ```bash
  pyarmor obfuscate \
    --recursive \
    --output dist \
    foo.py
  ```

## Security

- PyArmor obfuscate each function separately, then obfuscate the whole module. At runtime restore the current called function will be restored, and obfuscate immediately after call is completed.
- Cross protection `_pytransform`
  a dynamic library generated at obfuscate time.
    - Checked (checksum) by the obfuscated code
    - JIT mechanism
      - Define an instruction set base on GNU lightning.
      - Generate core functions by instruction set in C.
    - Other protections, at runtime, `_pytransform.so` will generate code from function 1 and run it with several protection approach.
      - Run function 1:
        - check codesum of code segment, if not expected, quit
        - check tickcount, if too long, quit
        - check there is any debugger, if found, quit
        - clear hardware breakpoints if possible
        - restore next function function 2
      - Run function 2... same as step 1.
- Limitations of free trial
    - The maximum size of code object is 35728 bytes in trial version
    - The scripts obfuscated by trial version are not private. It means anyone could generate the license file which works for these obfuscated scripts.
    - Without permission the trial version may not be used for the Python scripts of any commercial product.
    - [Purchase (TWD1,387.69)](https://order.shareit.com/cart/add?vendorid=200089125&PRODUCT[300871197]=1)
- Obfuscation step
  - Compile python script to code object, save to file.
