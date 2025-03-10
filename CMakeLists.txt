cmake_minimum_required(VERSION 3.21)
project(quill-art LANGUAGES CXX)

# Enforce Ninja
if(NOT CMAKE_GENERATOR STREQUAL "Ninja")
    message(FATAL_ERROR "This project requires Ninja as the generator.")
endif()

# Set default to Debug mode
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose Debug or Release" FORCE)
endif()

# Select the appropriate toolchain
if(DEFINED TARGET_OS)
    if(TARGET_OS STREQUAL "Linux")
        set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/toolchains/linux.cmake)
    elseif(TARGET_OS STREQUAL "Windows")
        set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/toolchains/windows.cmake)
    elseif(TARGET_OS STREQUAL "MacOS")
        set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/toolchains/macos.cmake)
    else()
        message(FATAL_ERROR "Unsupported TARGET_OS: ${TARGET_OS}")
    endif()
endif()

# Enforce Clang
set(CMAKE_C_COMPILER clang)
set(CMAKE_CXX_COMPILER clang++)

# Enable C++20 Modules
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Find all source files (Glob for .cpp and .cppm)
file(GLOB_RECURSE SRC_FILES src/*.cpp)
file(GLOB_RECURSE IMGUI imgui/*.cpp)
file(GLOB_RECURSE SDL_FILES sdl3/*.cpp)

# Compile executable
add_executable(quill-art ${SRC_FILES})
add_library(imgui STATIC ${IMGUI})
find_package(Vulkan REQUIRED)
find_package(SDL3 REQUIRED)
include_directories(${Vulkan_INCLUDE_DIRS})
target_link_libraries(quill-art PRIVATE imgui ${Vulkan_LIBRARIES} SDL3::SDL3)

# Include headers
target_include_directories(quill-art PRIVATE lib/)
target_include_directories(imgui PRIVATE lib/)

# Set build flags
target_compile_options(quill-art PRIVATE -Wall -Wextra -pedantic)
