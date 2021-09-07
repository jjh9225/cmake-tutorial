# scikit-build 를 사용하여 CMake 파이썬 확장 만들기 

scikit-build 는 복잡할 수 있는 C++ 파이썬 확장 생성을 CMake 를 사용하여 플랫폼에 상관없이 일관된 방법으로 빌드할 수 있게 해준다....
(는 명목상 이유이고, 윈도우즈에서 visual studio 용 c++ 프로젝트/솔루션을 만들어주어 개발 및 디버깅이 편하게 이루어질 수 있는 점 때문에 쓴다)

## step 1  파이썬 환경 준비하기 

가상환경 만들고 activate 한 다음, 다음 명령으로 필요한 환경이 구성되도록 한다

```
> pip install -r requirements.txt 
``` 

| 위 `requirements.txt` 는 `pip freeze` 라는 명령으로 현재 가상환경에 설치된 모든 패키지 목록을 덤프한것. 
(`pip freeze > requirements.txt` 하여 만든것)

또는 `pip install scikit-build` 해서 설치해도 된다. 

cmake 가 설치되어 있지 않은경우에는 `pip install cmake` 하면 파이썬 환경에 cmake를 설치 해주는 패키지가 설치된다. 


## step 2  C++ 코드 구현 체 추가하기 

`cpp/hello.cpp` 위치에 다음의 내용을 추가 

```cpp
// Python includes
#include <Python.h>

// STD includes
#include <stdio.h>

//-----------------------------------------------------------------------------
static PyObject *hello_example(PyObject *self, PyObject *args)
{
  // Unpack a string from the arguments
  const char *strArg;
  if (!PyArg_ParseTuple(args, "s", &strArg))
    return NULL;

  // Print message and return None
  PySys_WriteStdout("Hello, %s!\n", strArg);
  Py_RETURN_NONE;
}

//-----------------------------------------------------------------------------
static PyObject *elevation_example(PyObject *self, PyObject *args)
{
  // Return an integer
  return PyLong_FromLong(21463L);
}

//-----------------------------------------------------------------------------
static PyMethodDef hello_methods[] = {
  {
    "hello",
    hello_example,
    METH_VARARGS,
    "Prints back 'Hello <param>', for example example: hello.hello('you')"
  },

  {
    "elevation",
    elevation_example,
    METH_VARARGS,
    "Returns elevation of Nevado Sajama."
  },
  {NULL, NULL, 0, NULL}        /* Sentinel */
};

//-----------------------------------------------------------------------------
static struct PyModuleDef hello_module_def = {
  PyModuleDef_HEAD_INIT,
  "_hello",
  "Internal \"_hello\" module",
  -1,
  hello_methods
};

PyMODINIT_FUNC PyInit__hello(void)
{
  return PyModule_Create(&hello_module_def);
}
```

위 코드는 `example` 및 `elevation` 이라는 2개의 함수를 `_hello` 라는 이름의 모듈에 추가한다. 
(왜 `hello` 가 아니라 `_hello` 로 했냐 하면, `hello` 라는 이름으로 파이썬 레벨의 패키지를 만들고, `_hello` 라는 내부 패키지로 c++ 파이썬 확장패키지를 외부에 노출 시키는 방식으로 해서, c++ 레벨의 구현과 python 레벨의 구현을 함께 hybrid로 패키지를 구성할 수 있기 때문에 이런식으로 한것임)

## step 3. 파이썬 패키지 구성하기 

`hello` 디렉토리를 만들고 `hello\__init__.py` 파일을 만든다(파이썬 패키지 구성의 기본 방식)

```py
from ._hello import hello
from ._hello import elevation
```

위의 내용으로 보면, `_hello` 라는 현재 디렉토리의 모듈에서 `hello` 및 `elevation` 이라는 심벌(여기서는 함수)을 외부에 노출하고 있다. 
(c++ 파이썬 확장을 빌드하기 전까지는 사실, `_hello` 라는 모듈이 현재 디렉토리에는 아직 없다)

## step 3. setup.py 및 CMakeLists.txt 만들기

scikit-build 가 제공하는 기능을 사용하여 `setup.py` 를 구성한다. 

```py
import sys

from skbuild import setup

setup(
    name="hello-cpp",
    version="1.2.3",
    description="a minimal example package (cpp version)",
    author='Mirero System',
    license="Commercial",
    packages=['hello']
)
```

위에서 중요한 점은 `from skbuild import setup` 이다. 통상의 파이썬용 `setup.py` 는 `from setuptools import setup` 처럼 `setuptools` 라는 표준 파이썬 라이브러리에서 `setup` 함수를 가져왔으나, 여기에서는 `skibuild` 즉, scikit build가 제공하는 CMake를 사용하기 편리하도록 수정된 `setup` 함수를 사용한다. 

위 스크립트는 빌드를 위한 명령행 도구 역할을 한다. `python setup.py --help` 해 보면 `setup` 함수가 명령행 도구로 지원하는 여러 기능을 확인할 수 있다. 

한편 `CMakeLists.txt` 는 다음과 같이 구성한다. 

```
cmake_minimum_required(VERSION 3.4.0)

# 이 파이썬 프로젝트(=모듈)의 명칭
# 윈도우즈에서는 hello 라는 이름의 visual studio 솔루션 파일이 생성된다. 
# 리눅스에서는 hello 라는 타겟을 가지는 Makefile 이 생성된다. 
project(hello)

# scikit build 가 제공하는 python 환경 찾는 모듈을 사용하여 필요한 구성이 추가되도록 한다
# - include 경로
# - link 경로 
# - link 대상
find_package(PythonExtensions REQUIRED)

# 소스코드 목록을 지칭하는 CMake 변수를 하나 만든다 
set(HELLO_SRC 
    cpp/hello.cpp
    # 여기에 추가적인 *.hpp, *.cpp 등의 소스코드가 들어올 수 있다. 
)

# `_hello` 이름의 라이브러리를 만드는 프로젝트를 생성한다. 
add_library(_hello MODULE ${HELLO_SRC})

# `_hello` 라이브러리를 python 확장으로 만든다(예: *.pyd 확장자, dll )
python_extension_module(_hello)

# `_hello` 가 빌드된 다음 설치시에는 현재 디렉토리 기준으로 `hello` 라는 이름의 폴더에 복사된다.
install(TARGETS _hello LIBRARY DESTINATION hello)
```

## step 4. 생성된 파일들 확인 

```
{소스코드 디렉토리}
│  CMakeLists.txt
│  README.md        <--- 지금 읽고 있는 파일
│  requirements.txt
│  setup.py
├─cpp
│      hello.cpp
├─hello
│     __init__.py
```


## step 5. 빌드 환경 생성 및 빌드하기

지금 읽고 있는 README.md 파일이 있는  `{소스코드 디렉토리}` 에서 `python setup.py build` 명령을 수행하면 scikit build 가 cmake 를 사용하여 `_skbuild` 라는 디렉토리에 빌드환경을 구성하고, 성공적으로 환경이 구성되면, 빌드까지 수행한다. 


수행해야 할 명령은, 통상 python 패키지를 빌드할때 사용하는  `python setup.py build` 명령으로 수행된다(`scikit-build` 패키지가 제공하는 `setup` 함수가 모든걸 알아서 해준다. `python setup.py --help` 를 치면, 통상 `setuptools` 가 지공하는 `setup` 함수와는 달리 CMake 관련된 여러가지 명령을 추가로 더 입력할 수 있음을 알 수 있다)


```
>python setup.py build 


--------------------------------------------------------------------------------
-- Trying "Ninja (Visual Studio 16 2019 x64 v142)" generator
--------------------------------
---------------------------
----------------------
-----------------
------------
-------
--
Not searching for unused variables given on the command line.
-- The C compiler identification is MSVC 19.29.30038.1
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - failed
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2019/Professional/VC/Tools/MSVC/14.29.30037/bin/Hostx86/x64/cl.exe
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2019/Professional/VC/Tools/MSVC/14.29.30037/bin/Hostx86/x64/cl.exe - broken
CMake Error at D:/workspace/prj/study/python/scikit-build-sample-projects/.venv/Lib/site-packages/cmake/data/share/cmake-3.21/Modules/CMakeTestCCompiler.cmake:69 (message):
  The C compiler

    "C:/Program Files (x86)/Microsoft Visual Studio/2019/Professional/VC/Tools/MSVC/14.29.30037/bin/Hostx86/x64/cl.exe"

  is not able to compile a simple test program.

  It fails with the following output:

    Change Dir: D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp-python/_cmake_test_compile/build/CMakeFiles/CMakeTmp

    Run Build Command(s):C:/dev/util/ninja.exe cmTC_f3abb && [1/2] Building C object CMakeFiles\cmTC_f3abb.dir\testCCompiler.c.obj

    
... 중략 ....


--------------------------------
-- Trying "Visual Studio 16 2019 x64 v142" generator - success
--------------------------------------------------------------------------------

Configuring Project
  Working directory:
    D:\workspace\prj\study\python\scikit-build-sample-projects\hello-cpp-python\_skbuild\win-amd64-3.7\cmake-build
  Command:
    cmake 'D:\workspace\prj\study\python\scikit-build-sample-projects\hello-cpp-python' -G 'Visual Studio 16 2019' '-DCMAKE_INSTALL_PREFIX:PATH=D:\workspace\prj\study\python\scikit-build-sample-projects\hello-cpp-python\_skbuild\win-amd64-3.7\cmake-install' '-DPYTHON_EXECUTABLE:FILEPATH=d:\workspace\prj\study\python\scikit-build-sample-projects\.venv\Scripts\python.exe' -DPYTHON_VERSION_STRING:STRING=3.7.4 '-DPYTHON_INCLUDE_DIR:PATH=C:\Users\joonhwan.lee\AppData\Local\Programs\Python\Python37\Include' '-DPYTHON_LIBRARY:FILEPATH=C:\Users\joonhwan.lee\AppData\Local\Programs\Python\Python37\libs\python37.lib' -DSKBUILD:INTERNAL=TRUE '-DCMAKE_MODULE_PATH:PATH=d:\workspace\prj\study\python\scikit-build-sample-projects\.venv\lib\site-packages\skbuild\resources\cmake' -T v142 -A x64 -DCMAKE_BUILD_TYPE:STRING=Release

-- Selecting Windows SDK version 10.0.19041.0 to target Windows 10.0.19042.
_modinit_prefix:PyInit_
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp-python/_skbuild/win-amd64-3.7/cmake-build
.NET Framework용 Microsoft (R) Build Engine 버전 16.10.2+857e5a733
Copyright (C) Microsoft Corporation. All rights reserved.

  Checking Build System
  Building Custom Rule D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp-python/CMakeLists.txt
  hello.cpp
     D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp-python/_skbuild/win-amd64-3.7/cmake-build/Release/_hello.lib 라이브러리 및 D:/workspace/prj/study/python/scikit-build-sample-projects/h
  ello-cpp-python/_skbuild/win-amd64-3.7/cmake-build/Release/_hello.exp 개체를 생성하고 있습니다.
  _hello.vcxproj -> D:\workspace\prj\study\python\scikit-build-sample-projects\hello-cpp-python\_skbuild\win-amd64-3.7\cmake-build\Release\_hello.cp37-win_amd64.pyd
  Building Custom Rule D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp-python/CMakeLists.txt
  -- Install configuration: "Release"
  -- Installing: D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp-python/_skbuild/win-amd64-3.7/cmake-install/hello/_hello.cp37-win_amd64.pyd

copying hello\__init__.py -> _skbuild\win-amd64-3.7\cmake-install\hello\__init__.py

running build
running build_py
creating _skbuild\win-amd64-3.7\setuptools\lib
creating _skbuild\win-amd64-3.7\setuptools\lib\hello
copying _skbuild\win-amd64-3.7\cmake-install\hello\__init__.py -> _skbuild\win-amd64-3.7\setuptools\lib\hello
copying _skbuild\win-amd64-3.7\cmake-install\hello\_hello.cp37-win_amd64.pyd -> _skbuild\win-amd64-3.7\setuptools\lib\hello
copied 1 files
running build_ext
```


## step 6. 빌드 환경 확인하기 

`_skbuild\win-amd64-3.7\cmake-build` 디렉토리에 가면, `hello.sln` 이란 이름의 visual studio 솔루션 파일이 있음을 알 수 있다. 

`hello` 라는 이름은  CMakeLists.txt  스크립트의 

```
cmake_minimum_required(VERSION 3.4.0)
.
.
.
project(hello) <-----------  이 부분!!!!!이 solution 파일명이 된다.

.
.
.
```
에 의해 정해진것. 

해당 솔루션 파일을 visual studio에서 열면, CMakeLists.txt의 다음 부분

```
.
.
# `_hello` 이름의 라이브러리를 만드는 프로젝트를 생성한다. 
add_library(_hello MODULE ${HELLO_SRC})  <--- 여기
.
.
.

```
에 의해 `_hello` 라는 이름의 프로젝트가 열린다. 

이 프로젝트 뿐 아니라, 

- `ALL_BUILD` 프로젝트 : 전체 빌드를 하게 해주는 프로젝트. (현재는 프로젝트가 `_hello` 밖에 없지만, 여러개의 프로젝트가 있는 경우에 편리하게 전체 빌드용으로 사용할 수 있다)
- `INSTALL` 프로젝트 : 이걸 빌드하면, 생성된 바이너리가 특정 위치로 복사된다(통상 `_skibuild/win-amd64-{파이썬 버젼}/cmake-install/{모듈명}` 위치로 복사된다)
- `ZERO_CHECK` 프로젝트 : CMakeLists.txt 파일의 변경이 생긴 경우, Visual Studio의 솔루션 및 프로젝트 파일을 다시 생성하는 역할을 한다. 프로젝트 파일이 새로이 생성되는 경우 Visual Studio에서는 "프로젝트를 다시 여시겠습니까?" 와 같은 대화상자가 표시되고, "예"를 선택하여 다시 로딩되도록 할 수 있다. 

또한 솔루션은 다음의 설정을 가진다. 

- Debug : 디버그 빌드
- RelWithDebInfo : Release 빌드이지만, 디버깅 정보를 포함하는 pdb 파일이 함께 생성
- Release : 통상 배포시 사용 
- MinSizeRel : Release 빌드이면서 동시에 빌드시 바이너리 크기가 최소화되도록 하는 빌드. 

주로 Debug 와 Release 를 사용한다. (`scikit-build` 로 만들어진 경우, `pip install -e . ` 명령 수행시에는 Release 빌드가 사용된다)


## step 7. 개발환경(=현재의 파이썬 가상환경)에 `hello` 모듈 설치하기 

`pip install -e .` 명령을 수행하면, 현재 파이썬 환경에 `hello` 모듈이 directory link 로 설치되어, 개발시 `_hello` 바이너리가 변경될 때마다, 
갱신된 버젼이 인식된다(이를 "개발자 모드 설치" 라고 한다)

단, `_hello` 모듈이 사용중인 경우 빌드가 되지 않으므로, `hello` 를 통해 `_hello` 모듈이 사용되고 있는 모든 python 프로세스를 종료 후에 명령을 수행해야 한다. 

다음은 `hello` 패키지를 개발자 모드로 설치하고, 파이썬 인터프리터에서 `_hello` 모듈이 제공하는 기능이 정상적으로 실행되는 지 확인해 보자.

```
> pip install -e .
Obtaining file:///D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp-python
Installing collected packages: hello-cpp
  Running setup.py develop for hello-cpp
Successfully installed hello-cpp-1.2.3

> python 
Python 3.7.4 (tags/v3.7.4:e09359112e, Jul  8 2019, 20:34:20) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import hello
>>> dir(hello)
['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__path__', '__spec__', '_hello', 'elevation', 'hello']
>>> hello.hello('mirero')
Hello, mirero!
>>> hello.elevation()
21463
>>>
```


끝.

