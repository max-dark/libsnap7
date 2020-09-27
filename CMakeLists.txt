cmake_minimum_required(VERSION 3.15)

project(snap7)

# Snap7 paths and sources

# main lib
set(SNAP7_SRC_DIR "src")
set(SNAP7_SRC_CORE_DIR "${SNAP7_SRC_DIR}/core")
set(SNAP7_SRC_SYS_DIR "${SNAP7_SRC_DIR}/sys")
set(SNAP7_SRC_LIB_DIR "${SNAP7_SRC_DIR}/lib")
set(SNAP7_INCLUDE_DIRS
        "${SNAP7_SRC_CORE_DIR}"
        "${SNAP7_SRC_SYS_DIR}"
        "${SNAP7_SRC_LIB_DIR}"
        )

# Wrappers

set(SNAP7_WRAPPERS_DIR "release/Wrappers")
set(SNAP7_CCXX_WRAPPER_DIR "${SNAP7_WRAPPERS_DIR}/c-cpp")

# OS Detect

set(SNAP7_SYS_DEPS)

if(WIN32)
    list(APPEND SNAP7_SYS_DEPS ws2_32 winmm)
elseif(APPLE)
    list(APPEND SNAP7_SYS_DEPS pthread)
elseif(UNIX)
    list(APPEND SNAP7_SYS_DEPS pthread rt)
else()
    message(FATAL_ERROR "Unknown OS: ${CMAKE_SYSTEM_NAME}")
endif()

# Snap7 system

add_library(snap7_sys STATIC)
target_sources(snap7_sys
    PRIVATE
        "${SNAP7_SRC_SYS_DIR}/snap_msgsock.cpp"
        "${SNAP7_SRC_SYS_DIR}/snap_sysutils.cpp"
        "${SNAP7_SRC_SYS_DIR}/snap_tcpsrvr.cpp"
        "${SNAP7_SRC_SYS_DIR}/snap_threads.cpp"

    PUBLIC
        "${SNAP7_SRC_SYS_DIR}/snap_msgsock.h"
        "${SNAP7_SRC_SYS_DIR}/snap_platform.h"
        "${SNAP7_SRC_SYS_DIR}/snap_sysutils.h"
        "${SNAP7_SRC_SYS_DIR}/snap_tcpsrvr.h"
        "${SNAP7_SRC_SYS_DIR}/snap_threads.h"
        "${SNAP7_SRC_SYS_DIR}/sol_threads.h"
        "${SNAP7_SRC_SYS_DIR}/unix_threads.h"
        "${SNAP7_SRC_SYS_DIR}/win_threads.h"

)

target_include_directories(snap7_sys PUBLIC ${SNAP7_SRC_SYS_DIR})

target_link_libraries(snap7_sys ${SNAP7_SYS_DEPS})

set_target_properties(snap7_sys PROPERTIES
        POSITION_INDEPENDENT_CODE ON)

# Snap7 core

add_library(snap7_transport STATIC)

target_sources(snap7_transport
    PRIVATE
        "${SNAP7_SRC_CORE_DIR}/s7_isotcp.cpp"
        "${SNAP7_SRC_CORE_DIR}/s7_text.cpp"
    PUBLIC
        "${SNAP7_SRC_CORE_DIR}/s7_firmware.h"
        "${SNAP7_SRC_CORE_DIR}/s7_types.h"
        "${SNAP7_SRC_CORE_DIR}/s7_isotcp.h"
        "${SNAP7_SRC_CORE_DIR}/s7_text.h"
)
target_include_directories(snap7_transport PUBLIC ${SNAP7_SRC_CORE_DIR})
target_link_libraries(snap7_transport PUBLIC snap7_sys)
set_target_properties(snap7_transport PROPERTIES
        POSITION_INDEPENDENT_CODE ON)

add_library(snap7_micro_client STATIC)
target_sources(snap7_micro_client
    PRIVATE
        "${SNAP7_SRC_CORE_DIR}/s7_micro_client.cpp"
        "${SNAP7_SRC_CORE_DIR}/s7_peer.cpp"
    PUBLIC
        "${SNAP7_SRC_CORE_DIR}/s7_micro_client.h"
        "${SNAP7_SRC_CORE_DIR}/s7_peer.h"
)
target_include_directories(snap7_micro_client PUBLIC ${SNAP7_SRC_CORE_DIR})
target_link_libraries(snap7_micro_client PUBLIC snap7_transport)
set_target_properties(snap7_micro_client PROPERTIES
        POSITION_INDEPENDENT_CODE ON)

add_library(snap7_client STATIC)
target_sources(snap7_client
    PRIVATE
        "${SNAP7_SRC_CORE_DIR}/s7_client.cpp"
    PUBLIC
        "${SNAP7_SRC_CORE_DIR}/s7_client.h"
)
target_include_directories(snap7_client PUBLIC ${SNAP7_SRC_CORE_DIR})
target_link_libraries(snap7_client PUBLIC snap7_micro_client)
set_target_properties(snap7_client PROPERTIES
        POSITION_INDEPENDENT_CODE ON)

add_library(snap7_partner STATIC)
target_sources(snap7_partner
    PRIVATE
        "${SNAP7_SRC_CORE_DIR}/s7_partner.cpp"
    PUBLIC
        "${SNAP7_SRC_CORE_DIR}/s7_partner.h"
)
target_include_directories(snap7_partner PUBLIC ${SNAP7_SRC_CORE_DIR})
target_link_libraries(snap7_partner PUBLIC snap7_transport)
set_target_properties(snap7_partner PROPERTIES
        POSITION_INDEPENDENT_CODE ON)

add_library(snap7_server STATIC)
target_sources(snap7_server
    PRIVATE
        "${SNAP7_SRC_CORE_DIR}/s7_server.cpp"
    PUBLIC
        "${SNAP7_SRC_CORE_DIR}/s7_server.h"
)
target_include_directories(snap7_server PUBLIC ${SNAP7_SRC_CORE_DIR})
target_link_libraries(snap7_server PUBLIC snap7_transport)
set_target_properties(snap7_server PROPERTIES
        POSITION_INDEPENDENT_CODE ON)

# Snap7 C interface

set(SNAP7_LIB_SRC
        "${SNAP7_SRC_LIB_DIR}/snap7_libmain.cpp")

set(SNAP7_LIB_INCLUDES
        "${SNAP7_SRC_LIB_DIR}/snap7_libmain.h")

add_library(snap7 SHARED ${SNAP7_LIB_SRC} ${SNAP7_LIB_HEADERS})

target_include_directories(snap7 PUBLIC ${SNAP7_SRC_LIB_DIR})

target_link_libraries(snap7 PUBLIC snap7_client snap7_partner snap7_server)

# C Wrapper

add_library(snap7_ccxx STATIC
        "${SNAP7_CCXX_WRAPPER_DIR}/snap7.h")

target_link_libraries(snap7_ccxx snap7)
target_include_directories(snap7_ccxx PUBLIC "${SNAP7_CCXX_WRAPPER_DIR}")
set_target_properties(snap7_ccxx PROPERTIES
        POSITION_INDEPENDENT_CODE ON)

# C++ Wrapper

add_library(snap7++ STATIC
        "${SNAP7_CCXX_WRAPPER_DIR}/snap7.cpp")

target_link_libraries(snap7++ snap7_ccxx)
target_include_directories(snap7++ PUBLIC "${SNAP7_CCXX_WRAPPER_DIR}")
set_target_properties(snap7++ PROPERTIES
        POSITION_INDEPENDENT_CODE ON)
