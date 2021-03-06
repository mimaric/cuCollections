#=============================================================================
# Copyright (c) 2018-2020, NVIDIA CORPORATION.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#=============================================================================
cmake_minimum_required(VERSION 3.18 FATAL_ERROR)

project(cuCollections VERSION 0.0.1 LANGUAGES CXX CUDA)

###################################################################################################
# - build type ------------------------------------------------------------------------------------

# Set a default build type if none was specified
set(DEFAULT_BUILD_TYPE "Release")

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' since none specified.")
  set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

###################################################################################################
# - find packages we depend on --------------------------------------------------------------------

# Note: CUDA 10.2 debian packages are broken for cuda/std/atomic header as it is located in
# cuda/std/std/atomic
find_package(CUDAToolkit 10.2 REQUIRED)

include(cmake/Modules/CPM.cmake)

CPMFindPackage(
  NAME Thrust
  GITHUB_REPOSITORY NVIDIA/thrust
  GIT_TAG 1.10.0
  GIT_SHALLOW TRUE
)
thrust_create_target(Thrust)

CPMAddPackage(
    NAME libcudacxx
    GITHUB_REPOSITORY NVIDIA/libcudacxx
    GIT_TAG 1.3.0
    DOWNLOAD_ONLY YES
)

# TODO: Once libcu++ exports a target, use that instead
add_library(libcudacxx INTERFACE)
target_include_directories(libcudacxx INTERFACE "${libcudacxx_SOURCE_DIR}/include")


###################################################################################################
# - cuco target   ---------------------------------------------------------------------------------
add_library(cuco INTERFACE)
target_include_directories(cuco INTERFACE 
                           "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
                           $<INSTALL_INTERFACE:include>)
target_link_libraries(cuco INTERFACE libcudacxx CUDA::cudart Thrust CUB::CUB)
target_compile_features(cuco INTERFACE cxx_std_14 cuda_std_14)


# Build options
option(BUILD_TESTS "Configure CMake to build tests" ON)
option(BUILD_BENCHMARKS "Configure CMake to build (google) benchmarks" ON)
option(BUILD_EXAMPLES "Configure CMake to build examples" ON)

###################################################################################################
# - cmake modules ---------------------------------------------------------------------------------

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules/" ${CMAKE_MODULE_PATH})


###################################################################################################
# - optionally build tests ------------------------------------------------------------------------

if(BUILD_TESTS)
    add_subdirectory(tests)
endif(BUILD_TESTS)

###################################################################################################
# - Optionally build google benchmarks ------------------------------------------------------------

if(BUILD_BENCHMARKS)
    add_subdirectory(benchmarks)
endif(BUILD_BENCHMARKS)

###################################################################################################
# - Optionally build examples ---------------------------------------------------------------------

if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif(BUILD_EXAMPLES)
