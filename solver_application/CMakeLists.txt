cmake_minimum_required(VERSION 3.16)
project(solver LANGUAGES CXX)

add_executable(solver equation.cpp)
target_link_libraries(solver PRIVATE formatter_ex solver_lib)
target_compile_features(solver PRIVATE cxx_std_11)

install(TARGETS solver RUNTIME DESTINATION bin)
