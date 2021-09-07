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