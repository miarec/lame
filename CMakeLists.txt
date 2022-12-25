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

# Location of "config.h" file
include_directories(${CMAKE_CURRENT_BINARY_DIR}/cmake.config/private)

# --------------------------------------------------------
# mp3lame library
# --------------------------------------------------------
add_library(mp3lame)

list( APPEND SOURCES
   PRIVATE
      libmp3lame/VbrTag.c
      libmp3lame/bitstream.c
      libmp3lame/encoder.c
      libmp3lame/fft.c
      libmp3lame/gain_analysis.c
      libmp3lame/id3tag.c
      libmp3lame/lame.c
      libmp3lame/newmdct.c
      libmp3lame/presets.c
      libmp3lame/psymodel.c
      libmp3lame/quantize.c
      libmp3lame/quantize_pvt.c
      libmp3lame/reservoir.c
      libmp3lame/set_get.c
      libmp3lame/tables.c
      libmp3lame/takehiro.c
      libmp3lame/util.c
      libmp3lame/vbrquantize.c
      libmp3lame/version.c
)

if (${WITH_XMM})
    list( APPEND SOURCES
        PRIVATE
            libmp3lame/vector/xmm_quantize_sub.c
    )
endif()

target_sources(mp3lame PRIVATE ${SOURCES})

set_target_properties(mp3lame PROPERTIES
  POSITION_INDEPENDENT_CODE ON
)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/include/lame.h" 
    "${CMAKE_BINARY_DIR}/include/lame/lame.h"
    COPYONLY
)

target_include_directories(mp3lame
    PRIVATE 
        ${CMAKE_CURRENT_SOURCE_DIR}/libmp3lame
        ${CMAKE_CURRENT_SOURCE_DIR}/include
    PUBLIC 
        # where top-level project will look for the library's public headers
        $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/include>
        # where external projects will look for the library's public headers
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

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
  FILES "${CMAKE_BINARY_DIR}/include/lame/lame.h"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/lame"
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

