cmake_minimum_required(2.8)
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
include_directories(${CURRENT_CMAKE_BINARY_DIR})
