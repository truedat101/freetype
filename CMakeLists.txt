# CMakeLists.txt
#
# Copyright 2013-2015 by
# David Turner, Robert Wilhelm, and Werner Lemberg.
#
# Written originally by John Cary <cary@txcorp.com>
#
# This file is part of the FreeType project, and may only be used, modified,
# and distributed under the terms of the FreeType project license,
# LICENSE.TXT.  By continuing to use, modify, or distribute this file you
# indicate that you have read the license and understand and accept it
# fully.
#
#
# As a preliminary, create a compilation directory and change into it, for
# example
#
#   mkdir ~/freetype2.compiled
#   cd ~/freetype2.compiled
#
# Now you can say
#
#   cmake <path-to-freetype2-src-dir>
#
# to create a Makefile that builds a static version of the library.
#
# For a dynamic library, use
#
#   cmake <path-to-freetype2-src-dir> -D BUILD_SHARED_LIBS:BOOL=true
#
# For a framework on OS X, use
#
#   cmake <path-to-freetype2-src-dir> -D BUILD_FRAMEWORK:BOOL=true -G Xcode
#
# instead.
#
# For an iOS static library, use
#
#   cmake <path-to-freetype2-src-dir> -D IOS_PLATFORM=OS -G Xcode
#
# or
#
#   cmake <path-to-freetype2-src-dir> -D IOS_PLATFORM=SIMULATOR -G Xcode
#
# Please refer to the cmake manual for further options, in particular, how
# to modify compilation and linking parameters.
#
# Some notes.
#
# . `cmake' creates configuration files in
#
#     <build-directory>/include/freetype/config
#
#   which should be further modified if necessary.
#
# . You can use `cmake' directly on a freshly cloned FreeType git
#   repository.
#
# . `CMakeLists.txt' is provided as-is since it is normally not used by the
#   developer team.
#
# . If you want to disable the automatic generation of the distribution
#   targets, add the `-D FREETYPE_NO_DIST=true' command line argument.
#
# . Set the `WITH_ZLIB', `WITH_BZip2', `WITH_PNG', and `WITH_HarfBuzz'
#   CMake variables to `ON' or `OFF' to force or skip using a dependency.
#   Leave a variable undefined (which is the default) to use the dependency
#   only if it is available.  Example:
#
#     cmake ... -DWITH_ZLIB=ON -DWITH_HarfBuzz=OFF ...


cmake_minimum_required(VERSION 3.2)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.22.22.tar.gz"
    SHA1 "83f7dd182930dee784fafd29745f55e7cd06ca34"
)

include(CheckIncludeFile)

project(freetype VERSION 2.4.12)

# Compiler definitions for building the library
add_definitions(-DFT2_BUILD_LIBRARY)

hunter_add_package(ZLIB)
find_package(ZLIB CONFIG REQUIRED) # ZLIB::zlib

hunter_add_package(BZip2)
find_package(BZip2 CONFIG REQUIRED) # BZip2::bz2

hunter_add_package(PNG)
find_package(PNG CONFIG REQUIRED) # PNG::png

file(MAKE_DIRECTORY "${PROJECT_BINARY_DIR}/include/freetype/config")

if (UNIX)
  check_include_file("unistd.h" HAVE_UNISTD_H)
  check_include_file("fcntl.h" HAVE_FCNTL_H)
  check_include_file("stdint.h" HAVE_STDINT_H)

  file(READ "${PROJECT_SOURCE_DIR}/builds/unix/ftconfig.in"
    FTCONFIG_H)
  if (HAVE_UNISTD_H)
    string(REGEX REPLACE
      "#undef +(HAVE_UNISTD_H)" "#define \\1"
      FTCONFIG_H "${FTCONFIG_H}")
  endif ()
  if (HAVE_FCNTL_H)
    string(REGEX REPLACE
      "#undef +(HAVE_FCNTL_H)" "#define \\1"
      FTCONFIG_H "${FTCONFIG_H}")
  endif ()
  if (HAVE_STDINT_H)
    string(REGEX REPLACE
      "#undef +(HAVE_STDINT_H)" "#define \\1"
      FTCONFIG_H "${FTCONFIG_H}")
  endif ()
  string(REPLACE "/undef " "#undef "
    FTCONFIG_H "${FTCONFIG_H}")
  file(WRITE "${PROJECT_BINARY_DIR}/include/freetype/config/ftconfig.h"
    "${FTCONFIG_H}")
else ()
  file(READ "${PROJECT_SOURCE_DIR}/include/freetype/config/ftconfig.h"
    FTCONFIG_H)
  file(WRITE "${PROJECT_BINARY_DIR}/include/freetype/config/ftconfig.h"
    "${FTCONFIG_H}")
endif ()


# Create the options file

file(READ "${PROJECT_SOURCE_DIR}/include/freetype/config/ftoption.h"
  FTOPTION_H)
string(REGEX REPLACE
  "/\\* +(#define +FT_CONFIG_OPTION_SYSTEM_ZLIB) +\\*/" "\\1"
  FTOPTION_H "${FTOPTION_H}")
string(REGEX REPLACE
  "/\\* +(#define +FT_CONFIG_OPTION_USE_BZIP2) +\\*/" "\\1"
  FTOPTION_H "${FTOPTION_H}")
string(REGEX REPLACE
  "/\\* +(#define +FT_CONFIG_OPTION_USE_PNG) +\\*/" "\\1"
  FTOPTION_H "${FTOPTION_H}")

file(WRITE "${PROJECT_BINARY_DIR}/include/freetype/config/ftoption.h"
  "${FTOPTION_H}")

# Specify library include directories
include_directories("${PROJECT_SOURCE_DIR}/include")
include_directories(BEFORE "${PROJECT_BINARY_DIR}/include")

file(GLOB PUBLIC_HEADERS "include/ft2build.h" "include/freetype/*.h")
file(GLOB PUBLIC_CONFIG_HEADERS "include/freetype/config/*.h")
file(GLOB PRIVATE_HEADERS "include/freetype/internal/*.h")

set(BASE_SRCS
  src/autofit/autofit.c
  src/base/ftbase.c
  src/base/ftbbox.c
  src/base/ftbdf.c
  src/base/ftbitmap.c
  src/base/ftcid.c
  src/base/ftfstype.c
  src/base/ftgasp.c
  src/base/ftglyph.c
  src/base/ftgxval.c
  src/base/ftinit.c
  src/base/ftlcdfil.c
  src/base/ftmm.c
  src/base/ftotval.c
  src/base/ftpatent.c
  src/base/ftpfr.c
  src/base/ftstroke.c
  src/base/ftsynth.c
  src/base/ftsystem.c
  src/base/fttype1.c
  src/base/ftwinfnt.c
  src/bdf/bdf.c
  src/bzip2/ftbzip2.c
  src/cache/ftcache.c
  src/cff/cff.c
  src/cid/type1cid.c
  src/gzip/ftgzip.c
  src/lzw/ftlzw.c
  src/pcf/pcf.c
  src/pfr/pfr.c
  src/psaux/psaux.c
  src/pshinter/pshinter.c
  src/psnames/psnames.c
  src/raster/raster.c
  src/sfnt/sfnt.c
  src/smooth/smooth.c
  src/truetype/truetype.c
  src/type1/type1.c
  src/type42/type42.c
  src/winfonts/winfnt.c
)

if (WIN32)
  set(BASE_SRCS ${BASE_SRCS} builds/windows/ftdebug.c)
elseif (WINCE)
  set(BASE_SRCS ${BASE_SRCS} builds/wince/ftdebug.c)
else ()
  set(BASE_SRCS ${BASE_SRCS} src/base/ftdebug.c)
endif ()

set(CMAKE_DEBUG_POSTFIX d)

add_library(freetype
  ${PUBLIC_HEADERS}
  ${PUBLIC_CONFIG_HEADERS}
  ${PRIVATE_HEADERS}
  ${BASE_SRCS}
)

if (BUILD_SHARED_LIBS)
  set_target_properties(freetype PROPERTIES
    COMPILE_DEFINITIONS freetype_EXPORTS
  )
endif ()

target_include_directories(freetype
   PUBLIC $<INSTALL_INTERFACE:include/freetype2>
   )

target_link_libraries(freetype PUBLIC ZLIB::zlib)
target_link_libraries(freetype PUBLIC BZip2::bz2)
target_link_libraries(freetype PUBLIC PNG::png)

# Installations
# Note the trailing slash in the argument to the `DIRECTORY' directive
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
  DESTINATION include/freetype2
  PATTERN "internal" EXCLUDE
  PATTERN "ftconfig.h" EXCLUDE
  PATTERN "ftoption.h" EXCLUDE
)
install(FILES
  ${PROJECT_BINARY_DIR}/include/freetype/config/ftconfig.h
  ${PROJECT_BINARY_DIR}/include/freetype/config/ftoption.h
  DESTINATION include/freetype2/freetype/config
)
install(TARGETS freetype
  EXPORT "${TARGETS_EXPORT_NAME}"
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  FRAMEWORK DESTINATION Library/Frameworks
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${version_config}" COMPATIBILITY AnyNewerVersion
)

# Use 'TARGETS_EXPORT_NAME'
configure_package_config_file(
    "cmake/template/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
