#Pipline of pakaging python file to so/pyd(dll) file

##Environment preparation
###Linux
**Cython**: ```pip install Cython```
**GUN C Compiler(Gcc)**:```sudo apt-get install build-essential```
###Windows
**Cython**
**GUN C Compiler(Gcc)**
**Something related to VS**
##.so file  for linux
```python
# file to package: mytest.py
import datetime
class DataCenter():
    def gettime(self):
        print(datetime.datetime.now())
    def write_data(self):
        print("hello tester!")

# call file: so_test.py
from mytest import DataCenter
data = DataCenter()
data.gettime()
data.write_data()
```
Use`python so_test.py` will get following result showing the program works properly.
![image.png](https://s2.loli.net/2022/07/19/cNfPABox1dpZK7I.png)
```python
# package file: setup.py
from distutils.core import setup
from Cython.Build import cythonize
# [] contains the filename to package, multiple filenames are supported
setup(ext_modules = cythonize(["mytest.py"]))
```
Execute command `python setup.py build_ext` will generate folders `build` and file `mytest.c`, and .so file is in `build` folder. Put `so_test.py` file into path of .so file, then execute it and check the result.
```py
# directory tree
├── build
│   ├── lib.linux-x86_64-3.8
│   │   ├── mytest.cpython-38-x86_64-linux-gnu.so
        └── so_test.py
│   └── temp.linux-x86_64-3.8
│       └── mytest.o
```
Another `setup.py` generate .so file without platform_suffix:
```python
from distutils.core import setup
from Cython.Build import cythonize
from Cython.Distutils import build_ext
import os
from distutils import sysconfig

def get_ext_filename_without_platform_suffix(filename):
    name, ext = os.path.splitext(filename)
    ext_suffix = sysconfig.get_config_var('EXT_SUFFIX')

    if ext_suffix == ext:
        return filename

    ext_suffix = ext_suffix.replace(ext, '')
    idx = name.find(ext_suffix)

    if idx == -1:
        return filename
    else:
        return name[:idx] + ext
class BuildExtWithoutPlatformSuffix(build_ext):
    def get_ext_filename(self, ext_name):
        filename = super().get_ext_filename(ext_name)
        return get_ext_filename_without_platform_suffix(filename)
setup(ext_modules = cythonize(["mytest.py"]), cmdclass={'build_ext': BuildExtWithoutPlatformSuffix},)
```
```py
# directory tree
├── build
│   ├── lib.linux-x86_64-3.8
│   │   ├── mytest.so
│   │   └── so_test.py
│   └── temp.linux-x86_64-3.8
│       └── mytest.o
```
[Link](https://stackoverflow.com/questions/43982543/importerror-no-module-named-cython) may be helpful when facing "No module named Cython" even `pip install Cython` is executed.

##.pyd file for windows
`mytest.py` and `dll_test.py` are the same as above.
```python
#!/usr/bin/env python
#-*-coding:utf-8-*-
from distutils.core import setup
from Cython.Build import cythonsize
from distutils.extension import Extension

def main():
    # filename, can add multiple Extension('', [''])
    extensions = [Extension('mytest', ['mytest.py'])]
    setup(ext_modules=cythonsize(extensions))
if __name__ == '__main__':
    main()
```
