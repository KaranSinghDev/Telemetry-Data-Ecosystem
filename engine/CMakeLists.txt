# GTest integration requires a slightly newer CMake version for FetchContent.
cmake_minimum_required(VERSION 3.13)
project(insight_engine)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- CERN QA: Strict Compiler Flags ---
# Treat all warnings as errors.
add_compile_options(-Wall -Wextra -Werror -pedantic)

# --- Fetch and Configure Google Test (GTest) ---
include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip
)
# Force shared CRT to avoid MSVC/Windows specific issues
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)


# --- Define Our Project's Targets ---

# 1. The main shared library (our database engine)
# We compile it as a shared library (.so) so Python's ctypes can load it.
add_library(insight SHARED insight.cpp shard.cpp)
target_include_directories(insight PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

# 2. The test executable
add_executable(engine_test test_engine.cpp insight.cpp shard.cpp)
target_include_directories(engine_test PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

# --- Link GTest to Our Test Executable ---
target_link_libraries(engine_test PRIVATE GTest::gtest_main)
