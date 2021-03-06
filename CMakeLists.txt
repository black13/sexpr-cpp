# Written in 2017 by Henrik Steffen Gaßmann <henrik@gassmann.onl>
#
# To the extent possible under law, the author(s) have dedicated all
# copyright and related and neighboring rights to this software to the
# public domain worldwide. This software is distributed without any warranty.
#
# You should have received a copy of the CC0 Public Domain Dedication
# along with this software. If not, see
#
#     http://creativecommons.org/publicdomain/zero/1.0/
#
########################################################################
cmake_minimum_required(VERSION 3.2.2)
project(sexpr-cpp VERSION 0.0.0)

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
	message(FATAL_ERROR "in-source builds are not supported!")
endif()

if (CMAKE_C_COMPILER_ID STREQUAL "GNU"
    OR CMAKE_C_COMPILER_ID STREQUAL "Clang")
    set(GCC_COMPATIBLE 1)
endif()
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU"
    OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    set(GXX_COMPATIBLE 1)
endif()


##########################################################################
# library includes

# boost unit test framework
option(Boost_DEBUG "Use debug boost libraries" OFF)
option(Boost_USE_STATIC_LIBS "Use static boost libraries" ON)
find_package(Boost 1.60 REQUIRED COMPONENTS
	filesystem
	system
	unit_test_framework
)
add_definitions(-DBOOST_ALL_NO_LIB)
if (NOT Boost_USE_STATIC_LIBS)
	add_definitions(-DBOOST_ALL_DYN_LINK)
endif()


##########################################################################
# compiler adjustments

# adjust compiler warning levels
if(MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_SCL_SECURE_NO_WARNINGS)
    if(${CMAKE_C_FLAGS} MATCHES "/W[0-4]")
        string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
    else()
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W4")
    endif()
    if(${CMAKE_CXX_FLAGS} MATCHES "/W[0-4]")
        string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    else()
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W4")
    endif()
endif()
if (GCC_COMPATIBLE)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-long-long -Wpedantic -Wno-unknown-pragmas")
endif()
if (GXX_COMPATIBLE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -Wpedantic -Wno-unknown-pragmas")
endif()

# additional compiler tweaks
if (GCC_COMPATIBLE)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11")
endif()
if (GXX_COMPATIBLE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
endif()

if(WIN32)
    # specify winapi version
    add_definitions(-D_WIN32_WINNT=0x0601)
    # force unicode WINAPI symbols
	add_definitions(-DUNICODE -D_UNICODE)
endif()


########################################################################
# configure output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")


########################################################################
add_library(sexpr-cpp INTERFACE)
target_include_directories(sexpr-cpp INTERFACE include)

enable_testing()
add_subdirectory(tests)
