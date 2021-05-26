# Copyright (C) 2021, Xilinx Inc - All rights reserved
# Xilinx Resource Manger U30 Scaler Plugin 
                                    
# Licensed under the Apache License, Version 2.0 (the "License"). You may
# not use this file except in compliance with the License. A copy of the
# License is located at

#     http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations 
# under the License.

cmake_minimum_required(VERSION 2.8.11)
include (CMakeDependentOption)
project(xrmU30Scaler)
set(ENV{PKG_CONFIG_PATH} "$ENV{PKG_CONFIG_PATH}:/usr/lib/pkgconfig")

# Linking variables
find_package(PkgConfig REQUIRED)
pkg_check_modules(XRT REQUIRED xrt)
pkg_check_modules(XMA REQUIRED libxma2api)
pkg_check_modules(LIBXRM REQUIRED libxrm)

set (CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${XRT_LDFLAGS} --stats -Bsymbolic")

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_CXX_FLAGS	"${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wextra -O1 -g  -DDSA64 -fpermissive")

# Encoder Plugin
add_library(xrmU30Scaler SHARED
	filter-scaler-xrm-plg-u30.cpp
)

include_directories(.)

target_compile_options(xrmU30Scaler
	PUBLIC	${XRT_CFLAGS}
	PUBLIC	${XMA_CFLAGS}
        PUBLIC ${LIBXRM_CFLAGS}
)

target_include_directories(xrmU30Scaler
	PUBLIC ${XRT_INCLUDE_DIRS}/xma2
	PUBLIC ${XRT_INCLUDE_DIRS}
        PUBLIC ${LIBXRM_INCLUDE_DIRS}
)

# Set the location for library installation
install(TARGETS xrmU30Scaler DESTINATION ${CMAKE_INSTALL_PREFIX}/plugin)

# Packaging section
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_CONTACT "Rohit Athavale <rohit.athavale@xilinx.com>")
set(CPACK_PACKAGE_VENDOR "Xilinx")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Xilinx MPSoC XRM Scaler Plugin")
set(CPACK_PACKAGE_DESCRIPTION "The Xilinx MPSoC based XRM scaler plugin")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "xrt (>= 2.10.37) ")
set(CPACK_RPM_PACKAGE_DEPENDS "xrt >= 2.10.37 ")
set(CPACK_RPM_EXCLUDE_FROM_AUTO_FILELIST_ADDITION "/opt" "/opt/xilinx" "/usr/lib" "/usr/lib/pkgconfig")
include(CPack)
