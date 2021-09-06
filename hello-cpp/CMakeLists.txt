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