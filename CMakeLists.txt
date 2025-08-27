cmake_minimum_required(VERSION 3.15)
cmake_policy(SET CMP0091 OLD)
set(CMAKE_CXX_STANDARD 17)

project(test_packer CXX)
add_executable(test_packer test.cpp)