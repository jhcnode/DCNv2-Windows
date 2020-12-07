cmake_minimum_required(VERSION 3.4)
project(DCNv2)

find_package(CUDA QUIET REQUIRED)
find_package(Python COMPONENTS Development)
include(FindCUDA)
include(FindPython)

set(CMAKE_CXX_STANDARD 11)

#list(APPEND CMAKE_C_FLAGS "-std=c11 -O3 -ffast-math -Wno-unused-result")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} -use_fast_math)
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} -lineinfo)
#LIST(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_50,code=sm_50)    #for maxwell
#LIST(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_52,code=sm_52)    #for maxwell
LIST(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_60,code=sm_60)    #for pascal
LIST(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_61,code=sm_61)    #for pascal
LIST(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_62,code=sm_62)    #for jetson tx2
LIST(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_75,code=sm_75)    #for turing

LIST(APPEND CUDA_NVCC_FLAGS "-D__CUDA_NO_HALF_OPERATORS__")
LIST(APPEND CUDA_NVCC_FLAGS "-D__CUDA_NO_HALF2_OPERATORS__")
LIST(APPEND CUDA_NVCC_FLAGS "-D__CUDA_NO_HALF_CONVERSIONS__")

set(INCLUDE_DIRECTORIES
        ./src
        ${Python_RUNTIME_LIBRARY_DIRS}/lib/site-packages/torch/include
        ${Python_RUNTIME_LIBRARY_DIRS}/lib/site-packages/torch/torch/csrc/api/include
        ${Python_RUNTIME_LIBRARY_DIRS}/lib/site-packages/torch/include/TH
        ${Python_RUNTIME_LIBRARY_DIRS}/lib/site-packages/torch/include/THC
        )
include_directories(INCLUDE_DIRECTORIES)
#message(${INCLUDE_DIRECTORIES})


set(SOURCE_FILES
        src/cpu/dcn_v2_cpu.cpp
        src/cpu/dcn_v2_im2col_cpu.h
        src/cpu/dcn_v2_im2col_cpu.cpp
        src/cpu/dcn_v2_psroi_pooling_cpu.cpp
        src/cpu/vision.h
        src/cuda/dcn_v2_cuda.cu
        src/cuda/dcn_v2_im2col_cuda.cu
        src/cuda/dcn_v2_im2col_cuda.h
        src/cuda/dcn_v2_psroi_pooling_cuda.cu
        src/cuda/vision.h
        src/dcn_v2.h
)

add_library(cuda INTERFACE)
set_target_properties(cuda PROPERTIES
        INTERFACE_INCLUDE_DIRECTORIES ${CUDA_INCLUDE_DIRS}
        INTERFACE_LINK_LIBRARIES "${CUDA_LIBRARIES};${CUDA_curand_LIBRARY};${CUDA_CUFFT_LIBRARIES};${CUDA_CUBLAS_LIBRARIES};${CUDA_TOOLKIT_ROOT_DIR}/lib/x64/cudnn.lib"
        )
find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()


set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4819")
add_definitions(-D_CRT_RAND_S)
cuda_add_library(DCN32 ${SOURCE_FILES} src/vision.cpp)
target_link_libraries(DCN32 cuda)

