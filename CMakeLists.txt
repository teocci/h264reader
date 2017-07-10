cmake_minimum_required(VERSION 3.6)
project(h264reader)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(SOURCE_FILES
    bs.h
    h264_analyze.c
    h264_avcc.c
    h264_avcc.h
    h264_sei.c
    h264_sei.h
    h264_stream.c
    h264_stream.h)

add_executable(h264reader ${SOURCE_FILES})