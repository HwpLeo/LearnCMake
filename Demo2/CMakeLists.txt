cmake_minimum_required(VERSION 2.8)

project(Demo)

aux_source_directory(. DIR_SRCS)

add_executable(Demo2 ${DIR_SRCS})