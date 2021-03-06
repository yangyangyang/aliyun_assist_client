﻿#CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
# CMake 最低版本号要求
PROJECT (aliyun_installer) 

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(../third_party)
include_directories(../common)

include_directories(../third_party/libcurl/include)
link_directories(../third_party/libcurl/lib)

link_directories(../common)
add_definitions(-DCURL_STATICLIB)
if(WIN32)
    set(CMAKE_EXE_LINKER_FLAGS_RELEASE "/INCREMENTAL:NO /DEBUG" )
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_CXX_FLAGS_DEBUG   "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
    link_libraries(shlwapi Psapi version)
    link_libraries(ws2_32 Wldap32)
    link_libraries(libcurl.lib)
    add_definitions(-DUNICODE -D_UNICODE)
else(UNIX)
    include_directories(./linux)
    aux_source_directory(./linux DIR_SRCS)
    add_compile_options(-std=c++11)
if(DEFINED ENV{USE_STATIC_LINK})
    set (CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -I../third_party/glibc/include")
endif()
endif()
link_libraries(assist-common)
link_libraries(third_party)
aux_source_directory(. DIR_SRCS)
aux_source_directory(./packagemanager DIR_SRCS)
ADD_EXECUTABLE(aliyun_installer ${DIR_SRCS} ../VersionResource.rc)
if(WIN32)
SET_TARGET_PROPERTIES(aliyun_installer PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG   ${CMAKE_CURRENT_SOURCE_DIR}/../output/)
SET_TARGET_PROPERTIES(aliyun_installer PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_SOURCE_DIR}/../output/)
else(UNIX)
if(DEFINED ENV{USE_STATIC_LINK})
TARGET_LINK_LIBRARIES(aliyun_installer "-nostdlib -nostartfiles -static ../third_party/glibc/lib/crt1.o ../third_party/glibc/lib/crti.o   -Wl,--start-group ../third_party/glibc/lib/libpthread.a -lstdc++   ../third_party/glibc/lib/librt.a  ../third_party/glibc/lib/libdl.a  ../third_party/glibc/lib/libc.a  ../third_party/libcurl/lib/libcurl.a -lgcc  -lgcc_eh -Wl,--end-group  ../third_party/glibc/lib/crtn.o")
else()
TARGET_LINK_LIBRARIES(aliyun_installer -lcurl -lpthread -ldl)
endif()
SET_TARGET_PROPERTIES(aliyun_installer PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../output/)
endif()
