cmake_minimum_required(VERSION 3.13)

# Set the toolchain file before project()
if(NOT CMAKE_TOOLCHAIN_FILE)
    if(DEFINED ENV{PS2DEV})
        set(CMAKE_TOOLCHAIN_FILE "$ENV{PS2DEV}/cmake/ps2dev.cmake" CACHE FILEPATH "Toolchain file")
    else()
        message(FATAL_ERROR "PS2DEV environment variable is not set. Please set it to your PS2DEV installation path.")
    endif()
endif()

project(ps2stuff VERSION 1.0.0 LANGUAGES CXX C)

# Options
option(BUILD_TESTS "Build test projects" OFF)
option(DEBUG "Enable debug build" OFF)

# Set output library name
set(EE_LIB "libps2stuff.a")

# Include directories - just this project's own headers. Every real
# dependency (core/common's tamtypes.h, and any other packages/{core,
# world}/*/include) is already on the default compile flags the
# toolchain file (ps2dev.cmake) sets up for every EE target - no
# PS2SDK-specific include_directories()/link_directories() needed here
# (ps2dev.cmake used to define a PS2SDK variable pointing at a flat
# legacy tree that doesn't exist post-migration; this project doesn't
# need it).
include_directories(
    ${CMAKE_SOURCE_DIR}/include
)

# Compiler flags
if(DEBUG)
    add_compile_definitions(_DEBUG)
endif()

# Warning flags (matching Makefile)
set(WARNING_FLAGS
    -Wno-strict-aliasing
    -Wno-conversion-null
)

# VU0 code is broken so disable for now
add_compile_definitions(
    NO_VU0_VECTORS
    NO_ASM
)

add_compile_options(${WARNING_FLAGS})

# ============================================================================
# Source files
# ============================================================================
set(PS2STUFF_SOURCES
    src/core.cpp
    src/cpu_matrix.cpp
    src/displayenv.cpp
    src/drawenv.cpp
    src/eetimer.cpp
    src/gs.cpp
    src/gsmem.cpp
    src/imagepackets.cpp
    src/math.cpp
    src/matrix.cpp
    src/packet.cpp
    src/perfmon.cpp
    src/ps2stuff.cpp
    src/sprite.cpp
    src/texture.cpp
    src/timer.cpp
    src/utils.cpp
)

# ============================================================================
# Build the library
# ============================================================================
add_library(ps2stuff STATIC ${PS2STUFF_SOURCES})

set_target_properties(ps2stuff PROPERTIES
    OUTPUT_NAME "ps2stuff"
    ARCHIVE_OUTPUT_NAME "ps2stuff"
)

target_include_directories(ps2stuff PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# ============================================================================
# Install targets
# ============================================================================
# Relative to CMAKE_INSTALL_PREFIX (the caller passes
# -DCMAKE_INSTALL_PREFIX=$PS2DEV/packages/world/ps2stuff, matching the
# packages/<tier>/<name> layout every other package in this ecosystem
# uses) - not a hardcoded PS2SDK/ports path, which doesn't exist
# post-migration.
install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/${EE_LIB}
    DESTINATION lib
)

# package.yaml - so ps2build's own package tooling (list_packages,
# describe_package, resolve_libs) can see this package like any other
# installed under packages/<tier>/<name>, even though it's built by
# CMake rather than "ps2build install".
install(
    FILES ${CMAKE_SOURCE_DIR}/package.yaml
    DESTINATION .
)

install(
    DIRECTORY ${CMAKE_SOURCE_DIR}/include/ps2s
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)

# ============================================================================
# Include tests if enabled
# ============================================================================
if(BUILD_TESTS)
    add_subdirectory(tests)
endif()

# ============================================================================
# Print configuration summary
# ============================================================================
message(STATUS "")
message(STATUS "ps2stuff configuration:")
message(STATUS "  Version: ${PROJECT_VERSION}")
message(STATUS "  Debug build: ${DEBUG}")
message(STATUS "  Build tests: ${BUILD_TESTS}")
message(STATUS "  Output library: ${EE_LIB}")
message(STATUS "  Install prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "  VU0 vectors: DISABLED")
message(STATUS "  ASM optimizations: DISABLED")
message(STATUS "")
