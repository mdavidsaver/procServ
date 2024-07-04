cmake_minimum_required(VERSION 3.0)

project(procServ C CXX)

option(PROCSERV_DEVEL "Developer build" OFF)
option(PROCSERV_ACCESS_FROM_ANYWHERE
    "allow unauthenticated access from anywhere (CAVEAT: opens a security hole!!)"
    OFF
)
# systemd-utils
# doc

set(PROCSERV_EPICS_TOP ""
    CACHE PATH
    "create Makefile for EPICS build system: relative path to TOP"
)

if(CMAKE_COMPILER_IS_GNUCC AND PROCSERV_DEVEL)
    message(STATUS "Is developer build")
    set(CMAKE_BUILD_TYPE "DEBUG")
    add_compile_options("-Werror")
    add_definitions("-DDEBUG")
endif()

find_package(PkgConfig)
if(PKG_CONFIG_FOUND)
    pkg_check_modules(LIBTELNET libtelnet)
endif()

if(NOT LIBTELNET_FOUND)
    message(STATUS "Using bundled libtelnet")
    add_library(telnet STATIC
        libtelnet.c
        libtelnet.h
    )
    set(LIBTELNET_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
    set(LIBTELNET_LIBRARY telnet)
endif()
message(STATUS "LIBTELNET_INCLUDE_DIR=${LIBTELNET_INCLUDE_DIR}")
message(STATUS "LIBTELNET_LIBRARY=${LIBTELNET_LIBRARY}")

if(PROCSERV_ACCESS_FROM_ANYWHERE)
    message(STATUS "Allowing ALLOW_FROM_ANYWHERE")
    add_definitions("-DALLOW_FROM_ANYWHERE")
endif()

add_definitions("-DPACKAGE_STRING=\"procServ X.Y\"")

add_executable(procServ
    procServ.cc
    procServ.h
    connectionItem.cc
    acceptFactory.cc
    clientFactory.cc
    processFactory.cc
    processClass.h
)
target_link_libraries(procServ ${LIBTELNET_LIBRARY})
target_include_directories(procServ PRIVATE "${LIBTELNET_INCLUDE_DIR}")
