cmake_minimum_required(VERSION 3.21)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# iOS is arm64 only; Macs get both Apple silicon and Intel
if ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS" OR IOS)
    set(CMAKE_OSX_ARCHITECTURES "arm64")
else()
    set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")
endif()

project(InputBuffer VERSION 3.0.0)

file(GLOB SOURCES CONFIGURE_DEPENDS src/*.cpp)
add_library(${PROJECT_NAME} SHARED ${SOURCES})

if (NOT DEFINED ENV{GEODE_SDK})
    message(FATAL_ERROR "Unable to find Geode SDK! Please define GEODE_SDK environment variable to point to Geode")
endif()

add_subdirectory($ENV{GEODE_SDK} ${CMAKE_CURRENT_BINARY_DIR}/geode)

# Geode's iOS bindings define the FMOD functions the game has built in, but the generated
# source never includes FMOD's header, so FMOD_RESULT & co. are unknown there. Hand it the
# header (iOS only; the other platforms link FMOD and don't generate these).
if ((GEODE_TARGET_PLATFORM STREQUAL "iOS" OR IOS OR CMAKE_SYSTEM_NAME STREQUAL "iOS") AND TARGET GeodeBindings)
    target_compile_options(GeodeBindings PRIVATE "SHELL:-include fmod.h")
endif()

setup_geode_mod(${PROJECT_NAME})
