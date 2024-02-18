cmake_minimum_required(VERSION 3.18 FATAL_ERROR)

set(CMAKE_TOOLCHAIN_FILE $ENV{HOME}/vcpkg/scripts/buildsystems/vcpkg.cmake CACHE STRING "Vcpkg toolchain file")
set(CMAKE_CUDA_COMPILER "/usr/local/cuda/bin/nvcc"  CACHE PATH "CUDA COMPILER")

include(fetch_rapids.cmake)
include(rapids-cmake)
include(rapids-cpm)
include(rapids-cuda)
include(${rapids-cmake-dir}/cuda/set_runtime.cmake)
include(rapids-export)
include(rapids-find)
########################################################################################################################

rapids_cuda_init_architectures(libtorch)
project(libtorch LANGUAGES CXX CUDA)

SET(TORCH_CUDA_VERSION "cu121")
SET(TORCH_VERSION "2.2.0")

# Define the URL and the filename of the LibTorch distribution
set(LIBTORCH_URL "https://download.pytorch.org/libtorch/${TORCH_CUDA_VERSION}/libtorch-cxx11-abi-shared-with-deps-${TORCH_VERSION}%2B${TORCH_CUDA_VERSION}.zip")
set(LIBTORCH_ZIP "${CMAKE_BINARY_DIR}/libtorch.zip")

# Download LibTorch
if(NOT EXISTS "${LIBTORCH_ZIP}")
    message(STATUS "Downloading LibTorch...")
    file(DOWNLOAD "${LIBTORCH_URL}" "${LIBTORCH_ZIP}"
            SHOW_PROGRESS)
endif()

# Unzip LibTorch
set(LIBTORCH_UNZIP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/libtorch")
if(NOT EXISTS "${LIBTORCH_UNZIP_DIR}")
    message(STATUS "Unzipping LibTorch to ${LIBTORCH_UNZIP_DIR}...")
    execute_process(COMMAND ${CMAKE_COMMAND} -E tar xvf "${LIBTORCH_ZIP}"
            WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")
endif()

set(CAFFE2_USE_CUDNN ON)
set(CMAKE_CUDA_STANDARD 20)  # FOR LIBTORCH SAKE -> LEAVE AS 20
set(CMAKE_CUDA_STANDARD_REQUIRED ON)
set(Torch_DIR "${LIBTORCH_UNZIP_DIR}/share/cmake/Torch")

find_package(Torch REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

add_library(libtorch INTERFACE)
target_link_libraries(libtorch INTERFACE "${TORCH_LIBRARIES}")
set_property(TARGET libtorch PROPERTY CXX_STANDARD 20)
target_link_libraries(libtorch INTERFACE ${TORCH_LIBRARIES})

