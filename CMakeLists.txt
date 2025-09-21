cmake_minimum_required(VERSION 3.18)
project(pyovdb LANGUAGES CXX)

set(BUILD_SHARED_LIBS
    OFF
    CACHE BOOL "" FORCE)
set(OPENVDB_BUILD_SHARED
    OFF
    CACHE BOOL "" FORCE)
set(OPENVDB_USE_STATIC_DEPENDENCIES
    ON
    CACHE BOOL "" FORCE)
set(OPENVDB_BUILD_PYTHON_MODULE
    ON
    CACHE BOOL "" FORCE)

include(FetchContent)

# --- Blosc ---
FetchContent_Declare(
  blosc2
  GIT_REPOSITORY https://github.com/Blosc/c-blosc2.git
  GIT_TAG v2.21.2)
set(BUILD_SHARED
    OFF
    CACHE BOOL "" FORCE)
set(BUILD_STATIC
    ON
    CACHE BOOL "" FORCE)
set(BUILD_TESTS
    OFF
    CACHE BOOL "" FORCE)
set(BUILD_BENCHMARKS
    OFF
    CACHE BOOL "" FORCE)
set(DEACTIVATE_LZ4
    ON
    CACHE BOOL "" FORCE)
set(BUILD_BLOSC1X
    ON
    CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(blosc2)

# --- Boost ---
FetchContent_Declare(
  boost
  GIT_REPOSITORY https://github.com/boostorg/boost.git
  GIT_TAG boost-1.89.0)
set(BOOST_ENABLE_CMAKE
    ON
    CACHE BOOL "" FORCE)
set(BOOST_INCLUDE_LIBRARIES system iostreams thread chrono date_time regex)
set(BOOST_BUILD_SHARED
    OFF
    CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(boost)
execute_process(COMMAND ./bootstrap.sh WORKING_DIRECTORY ${boost_SOURCE_DIR})

execute_process(COMMAND ./b2 install --prefix=${CMAKE_BINARY_DIR}/boost-local
                WORKING_DIRECTORY ${boost_SOURCE_DIR})
set(BOOST_ROOT
    ${CMAKE_BINARY_DIR}/boost-local
    CACHE PATH "Local Boost install")
set(Boost_DIR
    ${BOOST_ROOT}/lib/cmake/Boost-1.89.0
    CACHE PATH "BoostConfig.cmake")

# --- Link Boost statically ---
set(Boost_USE_STATIC_LIBS
    ON
    CACHE BOOL "" FORCE)
set(Boost_NO_SYSTEM_PATHS
    ON
    CACHE BOOL "" FORCE)
set(Boost_INCLUDE_DIR ${boost_SOURCE_DIR})
set(Boost_LIBRARY_DIR ${boost_BINARY_DIR}/stage/lib)

# --- nanobind ---
FetchContent_Declare(
  nanobind
  GIT_REPOSITORY https://github.com/wjakob/nanobind.git
  GIT_TAG v2.9.2)
FetchContent_MakeAvailable(nanobind)
set(nanobind_DIR
    ${nanobind_SOURCE_DIR}/cmake
    CACHE PATH "Path to nanobind CMake config")

# --- jemalloc ---
FetchContent_Declare(
  jemalloc
  GIT_REPOSITORY https://github.com/jemalloc/jemalloc.git
  GIT_TAG 5.3.0)
FetchContent_MakeAvailable(jemalloc)
set(JEMALLOC_LIBRARY jemalloc)
set(JEMALLOC_INCLUDE_DIR ${jemalloc_SOURCE_DIR}/include)

# --- Add OpenVDB ---
add_subdirectory(upstream/openvdb)

install(TARGETS pyopenvdb LIBRARY DESTINATION ${SKBUILD_PROJECT_NAME})
