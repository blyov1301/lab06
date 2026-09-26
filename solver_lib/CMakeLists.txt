cmake_minimum_required(VERSION 3.16)
project(solver_lib LANGUAGES CXX)

add_library(solver_lib STATIC solver.cpp)
target_include_directories(solver_lib PUBLIC )
target_compile_features(solver_lib PUBLIC cxx_std_11)
install(TARGETS solver_lib ARCHIVE DESTINATION lib)
