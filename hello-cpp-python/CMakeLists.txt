cmake_minimum_required(VERSION 3.4.0)

project(hello)

find_package(PythonExtensions REQUIRED)

set(HELLO_SRC 
    hello/_hello.cpp
)

add_library(_hello MODULE ${HELLO_SRC})

python_extension_module(_hello)

install(TARGETS _hello LIBRARY DESTINATION hello)