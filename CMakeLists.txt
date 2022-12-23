cmake_minimum_required (VERSION 3.8)

option(BUILD_SHARED_LIBS "Build a shared library" OFF)

if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Release")
   message(STATUS "Build type not specified: default is Release")
endif()

#-------------------------------------------------------------------------------
# Read version number from version.h file
#-------------------------------------------------------------------------------
file(STRINGS "libmp3lame/version.h" VERSION)

string(REGEX MATCH "LAME_MAJOR_VERSION[ \t]+([0-9]+)" _ ${VERSION})
set(VERSION_MAJOR ${CMAKE_MATCH_1})

string(REGEX MATCH "LAME_MINOR_VERSION[ \t]+([0-9]+)" _ ${VERSION})
set(VERSION_MINOR ${CMAKE_MATCH_1})

string(REGEX MATCH "LAME_PATCH_VERSION[ \t]+([0-9]+)" _ ${VERSION})
set(VERSION_PATCH ${CMAKE_MATCH_1})

message(STATUS "Building version ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

project (lame
  VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH} 
  LANGUAGES C
)

# Identity of this package.
set(PACKAGE "${PROJECT_NAME}")
set(PACKAGE_NAME "${PROJECT_NAME}")
set(PACKAGE_TARNAME "${PROJECT_NAME}")
set(PACKAGE_VERSION "${PROJECT_VERSION}")
set(PACKAGE_STRING "${PROJECT_NAME} ${PROJECT_VERSION}")
set(PACKAGE_BUGREPORT "lame-dev@lists.sf.net")

# Generate config.h file
add_subdirectory(cmake.config)
add_compile_definitions(HAVE_CONFIG_H)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/cmake.config/private)
# include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

# --------------------------------------------------------
# Include projects from sub directorieds
# --------------------------------------------------------
add_subdirectory(libmp3lame)
