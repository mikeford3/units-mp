# The MIT License (MIT)
#
# Copyright (c) 2018 Mateusz Pusz
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required(VERSION 3.15)

# core library options
set(UNITS_DOWNCAST_MODE ON CACHE STRING "Select downcasting mode")
set_property(CACHE UNITS_DOWNCAST_MODE PROPERTY STRINGS AUTO ON OFF)

# find dependencies
find_package(gsl-lite CONFIG REQUIRED)

# check if libc++ is being used
include(CheckLibcxxInUse)
check_libcxx_in_use(UNITS_LIBCXX)

# core library definition
add_library(mp-units-core INTERFACE)
target_compile_features(mp-units-core INTERFACE cxx_std_20)
target_link_libraries(mp-units-core INTERFACE
    gsl::gsl-lite
)
target_include_directories(mp-units-core INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    if(UNITS_LIBCXX)
        find_package(range-v3 CONFIG REQUIRED)
        target_link_libraries(mp-units-core INTERFACE range-v3::range-v3)
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(mp-units-core INTERFACE
        /utf-8  # Specifies both the source character set and the execution character set as UTF-8
    )
endif()

if(DEFINED UNITS_DOWNCAST_MODE)
    set(downcast_mode_options OFF ON AUTO)
    list(FIND downcast_mode_options "${UNITS_DOWNCAST_MODE}" downcast_mode)
    if(downcast_mode EQUAL -1)
        message(FATAL_ERROR "'UNITS_DOWNCAST_MODE' should be one of ${downcast_mode_options} ('${UNITS_DOWNCAST_MODE}' received)")
    else()
        message(STATUS "UNITS_DOWNCAST_MODE: ${UNITS_DOWNCAST_MODE}")
        target_compile_definitions(mp-units-core INTERFACE UNITS_DOWNCAST_MODE=${downcast_mode})
    endif()
endif()

set_target_properties(mp-units-core PROPERTIES EXPORT_NAME core)
add_library(mp-units::core ALIAS mp-units-core)

# installation
install(TARGETS mp-units-core EXPORT mp-unitsTargets)
install(DIRECTORY include/units TYPE INCLUDE)
