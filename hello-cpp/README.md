# CMake 기초를 위한 프로젝트 구성하기

본 문서는 *윈도우즈 기준*으로  설명 한다. 

- 참고 자료 : https://www.kudryavka.me/?p=758
- 필수 준비사항 :
   - CMake 설치 : https://cmake.org/download/ 에서 `cmake-x.y.z-windows-x86_x64.*` 로 설치. `*.msi` 버젼은 쉬운 설치. `.zip` 은 압축해제후, `bin` 디렉토리를 `PATH` 환경변수로 잡는다. 
   - Visual Studio 2017 or 2019 설치(C++ 개발환경)

여기서는 2개의 C++ 를 프로젝트를 구성하는 3개의 소스코드를 만든다. 

## hello 프로젝트(정적 라이브러리 프로젝트)

다음의 2개 파일로 구성되는 라이브러리이다. 

### `hello.cpp`

```cpp
#include <iostream>
#include "hello.hpp"

void hello::say(){
    std::cout<<"hello world"<<std::endl;

}
```

### `hello.hpp`

```cpp
namespace hello{
    void say();
}
```

## main 프로젝트(애플리케이션(ex: *.exe) 프로젝트)

다음 1개의 코드로 구성. 여기서는 hello 라이브러리를 사용한다. 

### `main.cpp`

```cpp
#include "hello.hpp"

int main() {
    hello::say();
}
```

## CMakeList.txt 파일 만들기

```
#cmake 최소 버전 요구 사양
cmake_minimum_required(VERSION 3.8)

#프로젝트명 버전
project(myproject VERSION 1.0.0)

#라이브러리를 추가한다.
add_library(
    hello
    hello.hpp
    hello.cpp
)

#main.cpp의 파일을 가주고 main이라는 바이너리를 생성한다.
add_executable(main main.cpp)

#라이브러리 링킹
target_link_libraries(main PRIVATE hello)
```

## 지금까지 파일 구성 

```
{현재 디렉토리}
│  CMakeLists.txt
│  hello.cpp
│  hello.hpp
│  main.cpp
│  README.md  <--- 지금 보고 있는 README문서
```

## CMake 를 사용해 플랫폼 별 빌드파일 만들기 


현재디렉토리에서 빌드환경을 위한 디렉토리를 하나 만든다. 이를 테면 `_build` 라고 하자(이 디렉토리는 git 버젼 관리시 `.gitignore` 에 넣어야 한다)
그리고, `_build` 디렉토리로 이동한다.

```
> mkdir _build
> cd _build
```

이제 여기에서 `cmake .. ` 명령을 수행(이 명령의 의미는 `..` 즉, 상위디렉토리를 `CMakeLists.txt` 파일이 있는 소위 `소스 디렉토리` 로, 현재 디렉토리를 `빌드 디렉토리`로  빌드 환경을 생성한다는 의미)

```
> cmake ..
-- Building for: Visual Studio 16 2019
-- Selecting Windows SDK version 10.0.19041.0 to target Windows 10.0.19042.
-- The C compiler identification is MSVC 19.29.30038.1
-- The CXX compiler identification is MSVC 19.29.30038.1
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2019/Professional/VC/Tools/MSVC/14.29.30037/bin/Hostx64/x64/cl.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2019/Professional/VC/Tools/MSVC/14.29.30037/bin/Hostx64/x64/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp/_build
```

위 처럼 나오면, 현재 디렉토리에 `myproject.sln` 의 visual studio 2019 용 솔루션 파일이 생성된다. 즉, 빌드환경이 구성된것이다. 

여기서 실제로 빌드를 하려면.. 2가지 방법이 있다. 

- `cmake --build . --config Release` 명령을 수행 (`.` 즉 현재 디렉토리에서 빌드를 수행하되, `Release` 설정으로 빌드)
- `myproject.sln` 파일을 Visual Studio 2019 에서 열어서 빌드 

위 첫번째를 실행하면, 전체 Release 빌드가 수행된다. 

```
>cmake --build . --config Release               
.NET Framework용 Microsoft (R) Build Engine 버전 16.10.2+857e5a733
Copyright (C) Microsoft Corporation. All rights reserved.

  Checking Build System
  Building Custom Rule D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp/CMakeLists.txt
  hello.cpp
  hello.vcxproj -> D:\workspace\prj\study\python\scikit-build-sample-projects\hello-cpp\_build\Release\hello.lib
  Building Custom Rule D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp/CMakeLists.txt
  main.cpp
  main.vcxproj -> D:\workspace\prj\study\python\scikit-build-sample-projects\hello-cpp\_build\Release\main.exe
  Building Custom Rule D:/workspace/prj/study/python/scikit-build-sample-projects/hello-cpp/CMakeLists.txt
```
