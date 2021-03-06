#-------------------------------------------------------------------------------------------
# Copyright (C) Electronic Arts Inc.  All rights reserved.
#-------------------------------------------------------------------------------------------

#-------------------------------------------------------------------------------------------
# CMake info
#-------------------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.1)
project(EASTLTest CXX)
include(CTest)

#-------------------------------------------------------------------------------------------
# Include directories
#-------------------------------------------------------------------------------------------
include_directories(source "../include")

#-------------------------------------------------------------------------------------------
# Compiler Flags
#-------------------------------------------------------------------------------------------
set (CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${CMAKE_CURRENT_SOURCE_DIR}/../scripts/CMake")
include(CommonCppFlags)

# TODO: Tests fail to pass on mingw-w64 gcc 6 without strict aliasing disabled.
# The problem is likely a bug in the compiler that causes it to ignore EASTL_MAY_ALIAS
# on derived classes that are marked with EASTL_MAY_ALIAS.
# Needs investigating.
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU" AND MINGW)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-strict-aliasing")
endif()

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-pointer-bool-conversion -Wno-unknown-warning-option")
endif()

#-------------------------------------------------------------------------------------------
# Source files
#-------------------------------------------------------------------------------------------
file(GLOB EASTLTEST_SOURCES "source/*.cpp" "source/*.inl" "source/*.h")
set(SOURCES ${EASTLTEST_SOURCES})

#-------------------------------------------------------------------------------------------
# Defines
#-------------------------------------------------------------------------------------------
add_definitions(-D_CRT_SECURE_NO_WARNINGS)
add_definitions(-D_SCL_SECURE_NO_WARNINGS)
add_definitions(-DEASTL_OPENSOURCE=1)
add_definitions(-D_CHAR16T)
add_definitions(-DEASTL_THREAD_SUPPORT_AVAILABLE=0)

#-------------------------------------------------------------------------------------------
# Executable definition
#-------------------------------------------------------------------------------------------
add_executable(EASTLTest ${SOURCES})

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

set(EASTLTest_Libraries EASTL EATest)
if((NOT APPLE) AND (NOT WIN32))
    target_link_libraries(EASTLTest ${EASTLTest_Libraries} Threads::Threads rt)
else()
    target_link_libraries(EASTLTest ${EASTLTest_Libraries} Threads::Threads)
endif()

#-------------------------------------------------------------------------------------------
# Run Unit tests and verify the results.
#-------------------------------------------------------------------------------------------
add_test(EASTLTestRuns EASTLTest)
set_tests_properties (EASTLTestRuns PROPERTIES PASS_REGULAR_EXPRESSION "RETURNCODE=0")

