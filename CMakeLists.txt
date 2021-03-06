cmake_minimum_required (VERSION 3.1)

# set a default build type if none was provided
# this has to be done before the project() instruction!
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build (Debug or Release)" FORCE)
endif()

project (stitch)

set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")

# Detect the OS
if(${CMAKE_HOST_WIN32})
  set(WINDOWS 1)
elseif(${CMAKE_HOST_APPLE})
  set(MACOSX 1)
elseif(${CMAKE_HOST_UNIX})
  set(LINUX 1)
else()
  message(FATAL_ERROR "Unsupported operating system")
  return()
endif()

# Set some env vars
set (VERSION_MAJOR 0)
set (VERSION_MINOR 1)

if (MSVC)
  if(MSVC_VERSION LESS 1800) # 1700 = VS 2013
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
  endif()
  set(OBJ_EXT obj)
else()

  include(CheckCXXCompilerFlag)
  CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
  CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
  if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
  else()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
  endif()
  set(OBJ_EXT o)
endif()

#################################################################################################################
# ============================================ General build definitions ====================================== #
#################################################################################################################
find_package(Eigen3)
if(NOT EIGEN3_INCLUDE_DIR )
    message(FATAL_ERROR "Set EIGEN3_INCLUDE_DIR")
endif()
include_directories(${EIGEN3_INCLUDE_DIR})

if(MSVC)

  add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_DEPRECATE -DWIN32_LEAN_AND_MEAN -DVC_EXTRALEAN)

  # /Zo makes debug symbol in pdb in release mode in VS2015
  add_definitions(/fp:fast /GR- /Os /Zo /arch:AVX /openmp)
else()
  add_definitions(-O3 -g -msse3 -mavx -Wall -Wextra)
  if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    add_definitions(-fopenmp)
  endif()
endif()

add_subdirectory(src)
