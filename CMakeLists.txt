cmake_minimum_required(VERSION 3.14)
project(qubic CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Include the centralized compiler detection module
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(CompilerSetup)

# Build options
option(BUILD_TESTS "Build the test suite" ON)
option(BUILD_EFI "Build the EFI application" ON)
option(USE_SANITIZER "Build test with sanitizer support (clang only)" ON)

if(IS_WINDOWS)
    message(WARNING "Building on Windows using Visual Studio and CMake has undergone limited testing and is not officially supported at this time.")
    if(USE_SANITIZER)
        message(WARNING "Building tests with sanitizer support is not supported on Windows.")
    endif()
endif()

# Always include platform_common as it's needed for both application and tests
add_subdirectory(lib/platform_common)

if(BUILD_EFI OR BUILD_TESTS)
    add_subdirectory(lib/platform_efi)
endif()

# Build the tests first. On fail, the build will fail completely
if(BUILD_TESTS)
    message(STATUS "Building test suite")
    enable_testing()
    add_subdirectory(lib/platform_os)
    
    # If we're not building the application, we still need src for tests
    # but don't make it a default target
    if(NOT BUILD_EFI)
        add_subdirectory(src EXCLUDE_FROM_ALL)
    endif()
    
    add_subdirectory(test)
endif()

# Build the application if requested
if(BUILD_EFI)
    message(STATUS "Building EFI application")
    add_subdirectory(src)
endif()
