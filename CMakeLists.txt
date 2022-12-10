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

set(XRMMSCALER_PROJ "xilinx-u30-xrm-multiscaler")
set(XRMMSCALER_PROJ_VERSION "1.1.1")

find_program(UNAME_EXEC uname)
find_program(LSB_RELEASE_EXEC lsb_release)

execute_process(COMMAND ${LSB_RELEASE_EXEC} -is
    OUTPUT_VARIABLE LINUX_FLAVOR
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

if (${LINUX_FLAVOR} MATCHES "^(Ubuntu|Debian)")
        # amd64
        execute_process(
        COMMAND dpkg --print-architecture
        OUTPUT_VARIABLE CPACK_ARCH
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )
elseif (${LINUX_FLAVOR} MATCHES "^(RedHat|CentOS|Amazon|Fedora|SUSE)")
        # x86_64
        execute_process(
        COMMAND uname -m
        OUTPUT_VARIABLE CPACK_ARCH
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )
else()
        message( SEND_ERROR "Trying to build on incompatible system \"${LINUX_FLAVOR}\"." )
endif()

# Grab OS release short name
execute_process(COMMAND ${LSB_RELEASE_EXEC} -rs
        OUTPUT_VARIABLE LSB_RELEASE_REL_SHORT
        OUTPUT_STRIP_TRAILING_WHITESPACE)

if (${LINUX_FLAVOR} MATCHES "^Amazon")
        set(LSB_RELEASE_REL_SHORT "amzn${LSB_RELEASE_REL_SHORT}")
endif()

set(CPACK_PACKAGE_FILE_NAME "${XRMMSCALER_PROJ}_${XRMMSCALER_PROJ_VERSION}_${LSB_RELEASE_REL_SHORT}-${CPACK_ARCH}")
set(CPACK_PACKAGE_VERSION ${XRMMSCALER_PROJ_VERSION})
project(${XRMMSCALER_PROJ})

set(ENV{PKG_CONFIG_PATH} "$ENV{PKG_CONFIG_PATH}:/usr/lib/pkgconfig")

# Linking variables
find_package(PkgConfig REQUIRED)
pkg_check_modules(XMA REQUIRED libxma2api)
pkg_check_modules(LIBXRM REQUIRED libxrm)

set (CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} --stats -Bsymbolic")

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_CXX_FLAGS	"${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wextra -DDSA64 -fpermissive")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_FILE_LICENSE_PATH "/usr/share/doc/xilinx/u30")

# Encoder Plugin
add_library(xrmU30Scaler SHARED
	filter-scaler-xrm-plg-u30.cpp
)

include_directories(.)

target_compile_options(xrmU30Scaler
	PUBLIC	${XMA_CFLAGS}
        PUBLIC ${LIBXRM_CFLAGS}
)

target_include_directories(xrmU30Scaler
        PUBLIC ${LIBXRM_INCLUDE_DIRS}
)

# Set the location for library installation
install(TARGETS xrmU30Scaler DESTINATION ${CMAKE_INSTALL_PREFIX}/plugin)
install(FILES ${CPACK_RESOURCE_FILE_LICENSE} CONFIGURATIONS Release RUNTIME DESTINATION ${CPACK_FILE_LICENSE_PATH}/${XRMMSCALER_PROJ})

# Packaging section
set(CPACK_PACKAGE_CONTACT "Rohit Athavale <rohit.athavale@xilinx.com>")
set(CPACK_PACKAGE_VENDOR "Xilinx")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Xilinx MPSoC XRM Scaler Plugin")
set(CPACK_PACKAGE_DESCRIPTION "The Xilinx MPSoC based XRM scaler plugin")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "xrm (>= 1.3) ")
set(CPACK_RPM_PACKAGE_REQUIRES "xrm >= 1.3")
set(CPACK_RPM_EXCLUDE_FROM_AUTO_FILELIST_ADDITION "/opt" "/opt/xilinx" "/usr/lib" "/usr/lib/pkgconfig" "/usr/share/doc/xilinx" "/usr/share/doc/xilinx/u30")
include(CPack)
