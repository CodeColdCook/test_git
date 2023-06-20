# Minimum CMake required
cmake_minimum_required(VERSION 3.5)

project(protobuf_pmtd_msgs
  VERSION 1.0
  LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
add_definitions(-Wall -Wextra -Wno-unused-parameter -O2)
add_definitions(-O2)

# install path
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/install)

# Find required protobuf package
find_package(Protobuf  REQUIRED)
find_package(eCAL REQUIRED)

option(PROTO_GEN_CXX "build CXX protobuf file" OFF)

if(protobuf_VERBOSE)
  message(STATUS "Using Protocol Buffers ${protobuf_VERSION}")
endif()

message (STATUS "Project source dir : ${PROJECT_SOURCE_DIR}")

# Set c++ output path
SET(MESSAGE_DIR ${PROJECT_SOURCE_DIR}/output_cxx)
if(EXISTS "${MESSAGE_DIR}" AND IS_DIRECTORY "${MESSAGE_DIR}")
        SET(PROTO_META_BASE_DIR ${MESSAGE_DIR})
else()
        file(MAKE_DIRECTORY ${MESSAGE_DIR})
        SET(PROTO_META_BASE_DIR ${MESSAGE_DIR})
endif()

# set global value
set_property(GLOBAL PROPERTY PROTO_SENSOR_MSGS_DIR ${MESSAGE_DIR})

# set proto file path
LIST(APPEND PROTO_FLAGS -I${PROJECT_SOURCE_DIR}/proto)
#获取需要编译的proto文件
file(GLOB_RECURSE MSG_PROTOS ${PROJECT_SOURCE_DIR}/proto/*.proto)

set(MESSAGE_SRC "")
set(MESSAGE_HDRS "")

foreach(ABS_FIL ${MSG_PROTOS})
        file(RELATIVE_PATH REL_FIL ${PROJECT_SOURCE_DIR}/proto ${ABS_FIL})
        get_filename_component(FIL_WE ${ABS_FIL} NAME_WE)
        get_filename_component(DIR ${REL_FIL} DIRECTORY)

        # message(STATUS ${ABS_FIL})
        # message(STATUS ${DIR})

        list(APPEND MESSAGE_SRC "${MESSAGE_DIR}/${DIR}/${FIL_WE}.pb.cc")
        list(APPEND MESSAGE_HDRS "${MESSAGE_DIR}/${DIR}/${FIL_WE}.pb.h")

        IF(PROTO_GEN_CXX)
          execute_process(
                  COMMAND  ${PROTOBUF_PROTOC_EXECUTABLE} ${PROTO_FLAGS} --cpp_out=${PROTO_META_BASE_DIR} ${ABS_FIL}
                )
        ENDIF(PROTO_GEN_CXX)
endforeach()

set_source_files_properties(${MESSAGE_SRC} ${MESSAGE_HDRS} PROPERTIES GENERATED TRUE)

include_directories(${MESSAGE_DIR})

# useful macros
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/util.cmake)

# ==============#
# Build targets #
# ==============#

include(GenerateExportHeader)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

# shared libraries
if(NOT DEFINED BUILD_SHARED_LIBS)
  message(STATUS "${PROJECT_NAME}: Building dynamically-linked binaries")
  option(BUILD_SHARED_LIBS "Build dynamically-linked binaries" ON)
  set(BUILD_SHARED_LIBS ON)
endif()

# common library
zetton_cc_library(
  NAME
  ${PROJECT_NAME}
  SRCS
  ${MESSAGE_SRC}
  ${MESSAGE_HDRS}
  INCLUDES
  DEPS
  eCAL::core
  protobuf::libprotobuf)
