#   Copyright 2020
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.

cmake_minimum_required(VERSION 2.8.7)

project(webcfg)

include(ExternalProject)
include(CTest)

add_definitions(-std=c99)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -g -Werror -Wall -Wno-sizeof-pointer-memaccess -Wno-format-truncation -Wno-unused-parameter -D_GNU_SOURCE=1")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c99 -g -Werror -Wall -Wno-sizeof-pointer-memaccess -Wno-format-truncation -Wno-unused-parameter -D_GNU_SOURCE=1")

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
set(CMAKE_MACOSX_RPATH 1)
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -undefined dynamic_lookup")
endif()

set(INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}/_install)
set(PREFIX_DIR ${CMAKE_CURRENT_BINARY_DIR}/_prefix)
set(INCLUDE_DIR ${INSTALL_DIR}/include)
set(LIBRARY_DIR ${INSTALL_DIR}/lib)
set(LIBRARY_DIR64 ${INSTALL_DIR}/lib64)
set(COMMON_LIBRARY_DIR ${INSTALL_DIR}/lib/${CMAKE_LIBRARY_ARCHITECTURE})
set(TEST_RESULTS_DIR ${CMAKE_CURRENT_BINARY_DIR}/test_results)
file(MAKE_DIRECTORY ${TEST_RESULTS_DIR})

include_directories(${INCLUDE_DIR}
                    ${INCLUDE_DIR}/cjson
                    ${INCLUDE_DIR}/msgpack
		    ${INCLUDE_DIR}/trower-base64
		    ${INCLUDE_DIR}/wdmp-c
		    ${INCLUDE_DIR}/cimplog
			${INCLUDE_DIR}/wrp-c
#                    ${INCLUDE_DIR}/curl
                    )
if (FEATURE_SUPPORT_AKER)
include_directories(${INCLUDE_DIR}/nanomsg
		    ${INCLUDE_DIR}/libparodus
)
endif (FEATURE_SUPPORT_AKER)

if (WEBCONFIG_BIN_SUPPORT)

include_directories(${INCLUDE_DIR}/rbus
		    ${INCLUDE_DIR}/rtmessage
)

endif (WEBCONFIG_BIN_SUPPORT)

if (NOT BUILD_YOCTO)

if (MULTIPART_UTILITY)
add_definitions(-DMULTIPART_UTILITY)
endif (MULTIPART_UTILITY)


# msgpack-c external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(msgpack
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/msgpack
    GIT_REPOSITORY https://github.com/msgpack/msgpack-c.git
    GIT_TAG "7a98138f27f27290e680bf8fbf1f8d1b089bf138"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
                  -DMSGPACK_ENABLE_CXX=OFF
                  -DMSGPACK_BUILD_EXAMPLES=OFF
                  -DBUILD_TESTING=OFF
                  -DMSGPACK_BUILD_TESTS=OFF
)
add_library(libmsgpack STATIC SHARED IMPORTED)
add_dependencies(libmsgpack msgpack)

# base64 external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(trower-base64
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/trower-base64
    GIT_REPOSITORY https://github.com/Comcast/trower-base64.git
    GIT_TAG "fbb9440ae2bc1118866baefcea7ff814f16613dd"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR} -DBUILD_TESTING=OFF
)
add_library(libtrower-base64 STATIC SHARED IMPORTED)
add_dependencies(libtrower-base64 trower-base64)

# wdmp-c external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(wdmp-c
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/wdmp-c
    GIT_REPOSITORY https://github.com/Comcast/wdmp-c.git
    GIT_TAG "f9f687b6b4b10c2b72341e792a64334f0a409848"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
                  -DBUILD_TESTING=OFF
)
add_library(libwdmp-c STATIC SHARED IMPORTED)
add_dependencies(libwdmp-c wdmp-c)

# cimplog external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(cimplog
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/cimplog
    GIT_REPOSITORY https://github.com/Comcast/cimplog.git
    GIT_TAG "1.0.2"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libcimplog STATIC SHARED IMPORTED)
add_dependencies(libcimplog cimplog)

# cJSON external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(cJSON
    PREFIX ${PREFIX_DIR}/cJSON
    GIT_REPOSITORY https://github.com/DaveGamble/cJSON.git
    GIT_TAG "39853e5148dad8dc5d32ea2b00943cf4a0c6f120"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libcJSON STATIC IMPORTED)
add_dependencies(libcJSON cJSON)

if (NOT FEATURE_SUPPORT_MQTTCM)
# curl external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(curl
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/curl
    GIT_REPOSITORY https://github.com/curl/curl.git
    GIT_TAG "curl-7_64_1"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR} -DBUILD_TESTING=OFF
)
add_library(libcurl STATIC SHARED IMPORTED)
add_dependencies(libcurl curl)
endif()
#find_library( LIB_CURL,
#              NAMES curl
#              PATHS /usr/lib64 )

# wrp-c external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(wrp-c
	DEPENDS trower-base64 msgpack cimplog
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/wrp-c
    GIT_REPOSITORY https://github.com/Comcast/wrp-c.git
    GIT_TAG "1.0.1"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
                  -DMSGPACK_ENABLE_CXX=OFF
                  -DMSGPACK_BUILD_EXAMPLES=OFF
                  -DBUILD_TESTING=OFF
                  -DMAIN_PROJ_BUILD=ON
                  -DMAIN_PROJ_LIB_PATH=${LIBRARY_DIR}
		  -DMAIN_PROJ_INCLUDE_PATH=${INCLUDE_DIR}
)
add_library(libwrp-c STATIC SHARED IMPORTED)
add_dependencies(libwrp-c wrp-c)

if (FEATURE_SUPPORT_AKER)
# nanoMsg external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(nanomsg
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/nanomsg
    GIT_REPOSITORY https://github.com/nanomsg/nanomsg.git
    GIT_TAG "1.1.4"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libnanomsg STATIC SHARED IMPORTED)
add_dependencies(libnanomsg nanomsg)

# libparodus external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(libparodus
    DEPENDS trower-base64 msgpack nanomsg wrp-c
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/libparodus
    GIT_REPOSITORY https://github.com/Comcast/libparodus.git
    GIT_TAG "1.0.2"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR} -DBUILD_TESTING=OFF
	-DMAIN_PROJ_BUILD=ON
	-DMAIN_PROJ_LIB_PATH=${LIBRARY_DIR}
	-DMAIN_PROJ_LIB64_PATH=${LIBRARY_DIR64}
	-DMAIN_PROJ_COMMON_PATH=${COMMON_LIBRARY_DIR}
	-DMAIN_PROJ_INCLUDE_PATH=${INCLUDE_DIR}
)
add_library(liblibparodus STATIC SHARED IMPORTED)
add_dependencies(liblibparodus libparodus)
endif (FEATURE_SUPPORT_AKER)

if (BUILD_TESTING)
# cmocka external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(cmocka
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/cmocka
    GIT_REPOSITORY https://github.com/elventear/cmocka.git
    GIT_TAG "b71a3060699bc1a5b00e958be353772f42545ac2"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libcmocka STATIC SHARED IMPORTED)
add_dependencies(libcmocka cmocka)

endif (BUILD_TESTING)

#This RBUS already build from github actions rtrouted.sh script, So disabled in github workflow
if (( NOT BUILD_GIT) AND WEBCONFIG_BIN_SUPPORT)

# rbus external dependency
#-------------------------------------------------------------------------------
ExternalProject_Add(rbus
     PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/rbus
     GIT_REPOSITORY https://github.com/rdkcentral/rbus.git
     GIT_TAG main
     CMAKE_ARGS += -DBUILD_FOR_DESKTOP=ON -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR} -DBUILD_TESTING=OFF
)

add_library(librbuscore STATIC SHARED IMPORTED)
add_dependencies(librbuscore rbuscore)

add_library(librtMessage STATIC SHARED IMPORTED)
add_dependencies(librtMessage rtMessage)

add_library(librbus STATIC SHARED IMPORTED)
add_dependencies(librbus rbus)
endif ()
endif ()

link_directories ( ${LIBRARY_DIR} ${COMMON_LIBRARY_DIR} ${LIBRARY_DIR64} )


add_subdirectory(src)

if (BUILD_TESTING)
    add_subdirectory(tests)
endif (BUILD_TESTING)
