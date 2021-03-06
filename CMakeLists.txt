# by Mark Hill
cmake_minimum_required(VERSION 3.1 FATAL_ERROR)
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()
project(cs-usrlnd)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}
    ${CMAKE_SOURCE_DIR}/cmake/
)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(TEST_EXECUTABLE_NAME ${PROJECT_NAME}-test)
set(SOURCE_ROOT ${CMAKE_SOURCE_DIR}/src)
set(INCLUDE_ROOT ${CMAKE_SOURCE_DIR}/include)
set(SCRIPTS_ROOT ${CMAKE_SOURCE_DIR}/scripts)
set(SUBDIRS comm drivers orientation telem)

include(EditorHelpers)
include(ConfigManagement)

set(EIGEN_INCLUDE_PATH external/eigen/Eigen)
if(NOT EXISTS ${EIGEN_INCLUDE_PATH})
    message(STATUS "Cloning submodules")
    execute_process(COMMAND git submodule update --init)
endif()
include_directories(${INCLUDE_ROOT} ${EIGEN_INCLUDE_PATH})

if(CMAKE_BUILD_TYPE EQUAL Release)
    set(CMAKE_C_FLAGS "-Wall -Wextra -O3")
else()
    include(CTest)
    set(CMAKE_C_FLAGS "-Wall -Wextra -g -O0 --coverage")
    add_subdirectory(${CMAKE_SOURCE_DIR}/external/unittest-cpp EXCLUDE_FROM_ALL)
    include_directories(${CMAKE_SOURCE_DIR}/external/unittest-cpp)
    enable_testing()
endif()
set(CMAKE_CXX_FLAGS ${CMAKE_C_FLAGS})
set (CMAKE_C_STANDARD 11)
set (CMAKE_CXX_STANDARD 11)

find_program(GCOV gcov)
if(GCOV)
    message(STATUS "Found gcov: ${GCOV}")
    add_custom_target(coverage COMMAND sh ${SCRIPTS_ROOT}/coverage.sh)
endif()

foreach(DIR ${SUBDIRS})
    add_subdirectory(src/${DIR})
endforeach(DIR)

set(CONFIGURED_ONCE TRUE CACHE INTERNAL "")
