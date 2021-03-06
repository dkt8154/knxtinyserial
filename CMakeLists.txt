cmake_minimum_required(VERSION 3.5)

# verbose makefile
set(CMAKE_VERBOSE_MAKEFILE OFF)

project(KnxTinySerialProject VERSION 1.1.0 LANGUAGES CXX)

# project version
message(STATUS "Project version: ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}")

# get git revision
find_package(Git)
if(GIT_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} rev-list --max-count=1 HEAD
        OUTPUT_VARIABLE GIT_REVISION
        ERROR_QUIET
        )
    if(NOT ${GIT_REVISION} STREQUAL "")
        string(STRIP ${GIT_REVISION} GIT_REVISION)
    endif()
    message(STATUS "Current git revision is ${GIT_REVISION}")
else()
    set(GIT_REVISION "unknown")
endif()

# include GNU standard dirs
include(GNUInstallDirs)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/Config.h.in"
  "${PROJECT_BINARY_DIR}/Config.h"
  )

# add the binary tree to the search path for include files
# so that we will find Config.h
include_directories("${PROJECT_BINARY_DIR}")

# check cmake version
if(${CMAKE_VERSION} VERSION_LESS "3.13.0")
   message("Please consider to switch to CMake 3.13.0")
else()
   message("Good! Your cmake is update!")
endif()

# enable support for c++11
set (CMAKE_CXX_STANDARD 11)

# add options
option(ONLY_SERIAL "Enable only serial compilation. Do not use KnxTinySerial." OFF)

if(ONLY_SERIAL)
    # add C define
    add_definitions(-DONLY_SERIAL)
endif(ONLY_SERIAL)

# check pkg-config support
find_package(PkgConfig REQUIRED)

# check boost
find_package(Boost 1.62 REQUIRED COMPONENTS filesystem program_options)

# check required libraries
pkg_check_modules(LIBSERIAL REQUIRED libserial)

# build a CPack driven installer package
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set (CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set (CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set (CPACK_SYSTEM_NAME "${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")
set (CPACK_SET_DESTDIR "ON") # absolute path install
include (CPack)


add_subdirectory(lib)
add_subdirectory(knxread)
add_subdirectory(knxsend)
add_subdirectory(serialtest)
