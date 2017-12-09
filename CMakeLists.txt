cmake_minimum_required(VERSION 3.8)
project(opengltest)

set(CMAKE_CXX_STANDARD 11)

set(CMAKE_INCLUDE_PATH C:/vcpkg/installed/x86-windows/include)

include_directories(${CMAKE_INCLUDE_PATH})

add_executable(opengltest
        opengltest/opengltest.cpp
        opengltest/LoadShader.cpp
        opengltest/stdafx.cpp
        opengltest/stdafx.h
        opengltest/targetver.h)

add_executable(Build_Run Build_Ctrl_F5.cpp)
add_executable(Run Ctrl_F5.cpp)

