﻿cmake_minimum_required(VERSION 3.10)

project (TestWrapper VERSION 0.99 DESCRIPTION "C++ CUDA Wrapper Test")

find_package(CUDA REQUIRED)
find_package(pybind11 REQUIRED)
find_package(PythonLibs REQUIRED)


#Adds the CUDA_CLEAN_DEPENDS target 
CUDA_BUILD_CLEAN_TARGET()



set(CUDA_VERBOSE_BUILD ON)
set(CUDA_SEPARABLE_COMPILATION ON)

set(CUDA_USE_STATIC_CUDA_RUNTIME ON)
set(Boost_USE_STATIC_LIBS ON)

#Sets the appropriate runtime library for the host-side code genererated by NVCC
#This is the crucial part
if(MSVC)
	set(XCOMPILER_OPT_DEBUG "/MDd") #later try using --shared and -g
	set(XCOMPILER_OPT_RELEASE "/MD")

#TODO add those runtime proprierties for gcc/linux

endif()

set(XCOMPILER_OPT_
	"$<$<CONFIG:Debug>:${XCOMPILER_OPT_DEBUG}>"
	"$<$<CONFIG:Release>:${XCOMPILER_OPT_RELEASE}>")

#When debugging on the host, debug also on the device

set(NVCC_DEBUG "-G") #; -DTHRUST_DEBUG;
set(NVCC_RELEASE "")


set(NVCC_GEN_OPT  
	"$<$<CONFIG:Debug>:${NVCC_DEBUG}>"
	"$<$<CONFIG:Release>:${NVCC_RELEASE}>")

#Set here your target architecture
CUDA_SELECT_NVCC_ARCH_FLAGS(ARCH_FLAGS Auto)

LIST(APPEND CUDA_NVCC_FLAGS ${ARCH_FLAGS})
LIST(APPEND CUDA_NVCC_FLAGS ${NVCC_GEN_OPT})
LIST(APPEND CUDA_NVCC_FLAGS -Xcompiler ${XCOMPILER_OPT_})
LIST(APPEND CUDA_NVCC_FLAGS  -lcuda;  -lcublas; -lcudart; -lcudadevrt; --cudart static;  -rdc=true; ) 

include_directories( ${PYTHON_INCLUDE_DIRS}  ${pybind11_INCLUDE_DIR} ${PROJECT_SOURCE_DIR}/inc ${CUDA_INCLUDE_DIRS})

#Check cuda flags
MESSAGE(STATUS "CUDA flags: ${CUDA_NVCC_FLAGS}")


cuda_add_library(vectorlib SHARED
					 vector.cu vector.cpp inc/vector.h inc/vector.cuh)




#Add CUBLAS because NVCC seems to auto-convert canonnical kernles (e.g. adding two vectors) to the CUBLAS implementation
#TODO investigate this behavior
CUDA_ADD_CUBLAS_TO_TARGET( vectorlib )
# ${Boost_LIBRARIES} ${PYTHON_LIBRARIES}
target_link_libraries(vectorlib ${PYTHON_LIBRARIES}   ${CUDA_LIBRARIES} ${CUDA_CUBLAS_LIBRARIES} )


set_target_properties(vectorlib PROPERTIES SUFFIX .pyd)
set_target_properties(vectorlib PROPERTIES PREFIX "")
