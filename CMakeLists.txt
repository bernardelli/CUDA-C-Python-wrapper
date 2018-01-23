﻿cmake_minimum_required(VERSION 3.10)


project (TestWrapper VERSION 0.99 DESCRIPTION "C++ CUDA Wrapper Test")

set(CUDA_SEPARABLE_COMPILATION ON)
set(CUDA_USE_STATIC_CUDA_RUNTIME ON)

find_package(CUDA REQUIRED)
find_package(Boost REQUIRED COMPONENTS Python REQUIRED Numpy REQUIRED)
find_package(PythonLibs REQUIRED)

set(CUDA_VERBOSE_BUILD ON)

CUDA_BUILD_CLEAN_TARGET()


add_definitions(-DBOOST_PYTHON_STATIC_LIB -DBOOST_NUMPY_STATIC_LIB)


set(Boost_USE_STATIC_LIBS ON)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)


CUDA_SELECT_NVCC_ARCH_FLAGS(ARCH_FLAGS 5.0)
LIST(APPEND CUDA_NVCC_FLAGS ${ARCH_FLAGS})
LIST(APPEND CUDA_NVCC_FLAGS   -lcudart; -lcudadevrt;--cudart static;  -rdc=true; -Xcompiler "/MDd"; ) 

include_directories( ${PYTHON_INCLUDE_DIRS}  ${Boost_INCLUDE_DIRS} ${PROJECT_SOURCE_DIR}/inc ${CUDA_INCLUDE_DIRS})


MESSAGE(STATUS "CUDA flags: ${CUDA_NVCC_FLAGS}")



cuda_add_library(vectorlib SHARED
					 vector.cu vector.cpp inc/vector.h)


CUDA_ADD_CUBLAS_TO_TARGET( vectorlib )



link_directories(${Boost_LIBRARY_DIRS} )


target_link_libraries(vectorlib  ${PYTHON_LIBRARIES}  ${Boost_LIBRARIES} ${CUDA_LIBRARIES} ${CUDA_CUBLAS_LIBRARIES} )


set_target_properties(vectorlib PROPERTIES SUFFIX .pyd)
set_target_properties(vectorlib PROPERTIES PREFIX "")