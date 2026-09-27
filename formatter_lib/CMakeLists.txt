cmake_minimum_required(VERSION 3.16)
project(formatter LANGUAGES CXX)

add_library(formatter STATIC formatter.cpp)
target_include_directories(formatter PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_compile_features(formatter PUBLIC cxx_std_11)

install(TARGETS formatter ARCHIVE DESTINATION lib)
