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

# ----------------------------------------------------------------------
# Installation
# ----------------------------------------------------------------------
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Library files
install(TARGETS mp3lame
  EXPORT mp3lameTargets
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
  INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

# Header files
install(
  FILES "${CMAKE_CURRENT_SOURCE_DIR}/include/lame.h"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

# ----------------------------------------------------------------------
# pkgconfig
# ----------------------------------------------------------------------
if(IS_ABSOLUTE "${CMAKE_INSTALL_LIBDIR}")
  set(PKGCONFIG_LIBDIR "${CMAKE_INSTALL_LIBDIR}")
else()
  set(PKGCONFIG_LIBDIR "\${prefix}/${CMAKE_INSTALL_LIBDIR}")
endif()
if(IS_ABSOLUTE "${CMAKE_INSTALL_INCLUDEDIR}")
  set(PKGCONFIG_INCLUDEDIR "${CMAKE_INSTALL_INCLUDEDIR}")
else()
  set(PKGCONFIG_INCLUDEDIR "\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}")
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake.packaging/libmp3lame.pc.in
  ${CMAKE_CURRENT_BINARY_DIR}/cmake.packaging/libmp3lame.pc @ONLY)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cmake.packaging/libmp3lame.pc DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

# ----------------------------------------------------------------------
# CMake config
# ----------------------------------------------------------------------
set(MP3LAME_INSTALL_CMAKEDIR "${CMAKE_INSTALL_LIBDIR}/cmake/mp3lame")

install(
  EXPORT mp3lameTargets
  FILE mp3lameTargets.cmake
  NAMESPACE mp3lame::
  DESTINATION "${MP3LAME_INSTALL_CMAKEDIR}"
)
configure_package_config_file(cmake.packaging/mp3lameConfig.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/cmake.packaging/mp3lameConfig.cmake"
  INSTALL_DESTINATION "${MP3LAME_INSTALL_CMAKEDIR}"
)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/cmake.packaging/mp3lameConfigVersion.cmake"
  VERSION "${CMAKE_PROJECT_VERSION}"
  COMPATIBILITY SameMajorVersion
)

install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/cmake.packaging/mp3lameConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake.packaging/mp3lameConfigVersion.cmake"
  DESTINATION "${MP3LAME_INSTALL_CMAKEDIR}"
)

