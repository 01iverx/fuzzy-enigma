cmake_minimum_required(3.5)
project(fuzzy-enigma)

#1.Protobuf
#1.1 Find protobuf requied package
find_package(Protobuf REQUIRED)
if(PROTOBUF_FOUND)
    message(STATUS "Protobuf library found")
else()
    message(FATAL ERROR "Protobuf library is needed but can not be found")
endif()

#1.2 Include directories for generating pb.h pb.cc
include_directories(${PROTOBUF_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_BINARY_DIR})
PROTOBUF_GEN_CPP(PROTO_SRCS PROTO_HDRS ${CMAKE_SOURCE_DIR/src/})

#1.3 Header file path
message("-- " ${PROTO_HDRS})
include_directories(${CMAKE_SOURCE_DIR}/include/protobuf)

#set build output
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/BuildOutput/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/BuildOutput/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/BuildOutput/bin)
include_directories(${CMAKE_SOURCE_DIR}/BuildOutput/lib)

#version info
option(WITH_BUILD_PUBLISH "" ON)
if(WITH_BUILD_PUBLISH)
    string(TIMESTAMP vDateTime "%Y-%m-%d %H:%M:%S")
    message("Timestamp " ${TIMESTAMP})
    
    #vProjectDir
    execute_process(
        COMMAND basename ${CMAKE_CURRENT_SOURCE_DIR}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE vProjectDir
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )

    #vGitBranch
    execute_process(
        COMMAND git rev-parse --abbrev-ref HEAD
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE vGitBranch
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )

    #vGitCommit
    execute_process(
        COMMAND git log -1 --pretty=format:%h
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE vGitCommit
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )

    set(vFileName "${CMAKE_CURRENT_SOURCE_DIR}/src/version.h")
    file(WRITE  ${vFileName} "#ifndef FUZZY_ENIGMA_VERSION_H\r\n#define FUZZY_ENIGMA_VERSION_H\r\n" )
    file(APPEND ${vFileName} "static const char* version=\"${vDateTime}:${vProjectDir}:${vGitBranch}:${vGitCommit}\";\r\n")
    file(APPEND ${vFileName} "#endif")
    
endif(WITH_BUILD_PUBLISH)

# set cxx_flags
add_definitions(-std=c++17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wshadow -Wpointer-arith -Wundef -Wredundant-decls")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -Wall -fPIC")

# Debug
set(CMAKE_C_FLAGS_DEBUG  "${CMAKE_C_FLAGS_DEBUG} -O0 -g -ggdb3")
set(CMAKE_CXX_FLAGS_DEBUG  "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g -ggdb3")

# Release
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -s -O2 ")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -s -O2 ")

if(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(CMAKE_BUILD_TYPE "Release")
else()
    set(CMAKE_BUILD_TYPE "Debug")
endif()

include_directories(src)

add_executable(${PROJECT_NAME}
        ${PROTO_SRCS}
        ${PROTO_HDRS}
)

target_link_libraries(${PROJECT_NAME}
        glog gflags
        pthread
        protobuf
        ${PROTOBUF_LIBRARIES}
)