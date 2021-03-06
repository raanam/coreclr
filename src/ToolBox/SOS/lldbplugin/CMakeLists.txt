project(sosplugin)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

add_definitions(-D_TARGET_AMD64_=1)
add_definitions(-DDBG_TARGET_64BIT=1)
add_definitions(-DDBG_TARGET_AMD64=1)
add_definitions(-DDBG_TARGET_WIN64=1)

set(ENABLE_LLDBPLUGIN ${CLR_CMAKE_PLATFORM_UNIX} CACHE BOOL "Enable building the SOS plugin for LLDB.")
set(REQUIRE_LLDBPLUGIN ${CLR_CMAKE_PLATFORM_LINUX} CACHE BOOL "Require building the SOS plugin for LLDB.")

set(WITH_LLDB_LIBS "" CACHE PATH "Path to LLDB libraries")
set(WITH_LLDB_INCLUDES "" CACHE PATH "Path to LLDB headers")

if(NOT ENABLE_LLDBPLUGIN)
    return()
endif()

# Check for LLDB library
find_library(LLDB NAMES lldb-3.6 lldb-3.5 LLDB lldb PATHS "${WITH_LLDB_LIBS}" PATH_SUFFIXES llvm)
if(LLDB STREQUAL LLDB-NOTFOUND)
    if(REQUIRE_LLDBPLUGIN)
        message(FATAL_ERROR "Cannot find lldb-3.5 or lldb-3.6. Try installing lldb-3.6-dev (or the appropriate package for your platform)")
    else()
        message(WARNING "Cannot find lldb-3.5 or lldb-3.6. Try installing lldb-3.6-dev (or the appropriate package for your platform)")
    endif()
    return()
endif()

message(STATUS "LLDB: ${LLDB}")

# Check for LLDB headers
find_path(LLDB_H "lldb/API/LLDB.h" PATHS "${WITH_LLDB_INCLUDES}")
if(LLDB_H STREQUAL LLDB_H-NOTFOUND)
    find_path(LLDB_H "lldb/API/LLDB.h" PATHS "/usr/lib/llvm-3.6/include")
    if(LLDB_H STREQUAL LLDB_H-NOTFOUND)
        find_path(LLDB_H "lldb/API/LLDB.h" PATHS "/usr/lib/llvm-3.5/include")
        if(LLDB_H STREQUAL LLDB_H-NOTFOUND)
            if(REQUIRE_LLDBPLUGIN)
                message(FATAL_ERROR "Cannot find LLDB.h. Try installing lldb-3.6-dev (or the appropriate package for your platform)")
            else()
                message(WARNING "Cannot find LLDB.h Try installing lldb-3.6-dev (or the appropriate package for your platform)")
            endif()
            return()
        endif()
    endif()
endif()

message(STATUS "LLDB_H: ${LLDB_H}")

add_compile_options(-Wno-delete-non-virtual-dtor)

include_directories(inc)
include_directories("${LLDB_H}")
include_directories(${CLR_DIR}/src/debug/inc)
include_directories(${CLR_DIR}/src/inc)

set(SOURCES
    sosplugin.cpp
    soscommand.cpp
    debugclient.cpp
)

add_library(sosplugin SHARED ${SOURCES})
add_dependencies(sosplugin sos)

if (CLR_CMAKE_PLATFORM_UNIX)
    target_link_libraries(sosplugin ${LLDB})
endif()

# add the install targets
install (TARGETS sosplugin DESTINATION .)
