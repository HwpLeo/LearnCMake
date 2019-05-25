cmake_minimum_required(VERSION 2.8)

project(Demo)

aux_source_directory(. DIR_SRCS)

add_subdirectory(math)

add_executable(Demo3 ${DIR_SRCS})

target_link_libraries(Demo3 MathFunctions)