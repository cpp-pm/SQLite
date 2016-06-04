#
# Copyright (c) 2008-2016 the Urho3D project.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
#

cmake_minimum_required(VERSION 3.0)

# Note: original TARGET_NAME is lower case, but top directory is upper case
project(sqlite VERSION 1.0.0)


# https://www.sqlite.org/compile.html
add_definitions (-DSQLITE_USE_URI=1 -DSQLITE_ENABLE_COLUMN_METADATA -DSQLITE_SOUNDEX)
# Do not use pthread and dl libraries for Web platform
if (WEB)
    add_definitions (-DSQLITE_THREADSAFE=0 -DSQLITE_OMIT_LOAD_EXTENSION)
endif ()

# Define source files
set (SOURCE_FILES src/sqlite3.c)
set (TARGET_PUBLIC_HDRS src/sqlite3.h src/sqlite3ext.h)

# Note: For initial hunter build we will only build the library:
# Setup additional SQLite CLI standalone target (this target can be transfered and executed on an embedded device, such as Raspberry Pi and Android)
# if (NOT IOS AND NOT WEB)
#     # Define target name for SQLite shell
#     set (TARGET_NAME isql)
#
#     # Define source files
#     set (SOURCE_FILES src/shell.c src/sqlite3.c src/sqlite3.h)
#
#     # Define dependency libs
#     if (NOT WIN32)
#         set (LIBS dl)
#     endif ()
#     if (READLINE_FOUND)
#         add_definitions (-DHAVE_READLINE)
#         list (APPEND INCLUDE_DIRS ${READLINE_INCLUDE_DIR})
#         list (APPEND LIBS ${READLINE_LIBRARIES})
#     endif ()
#
#     # Setup target
#     setup_executable (NODEPS)
# endif ()

# SQLite will not work correctly with the -ffast-math option of GCC
string (REPLACE -ffast-math "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")

add_library(sqlite ${SOURCE_FILES})

#### install 

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion
  )

configure_package_config_file(
  "cmake/Config.cmake.in"
  "${project_config}"
  INSTALL_DESTINATION "${config_install_dir}"
  )

install(
  TARGETS sqlite
  EXPORT "${targets_export_name}"
  LIBRARY DESTINATION "lib"
  ARCHIVE DESTINATION "lib"
  RUNTIME DESTINATION "bin"
  INCLUDES DESTINATION "${include_install_dir}"
  )

foreach(HEADER ${TARGET_PUBLIC_HDRS})
  install(
    FILES "${CMAKE_CURRENT_SOURCE_DIR}/${HEADER}"
    DESTINATION "${include_install_dir}/${PROJECT_NAME}"
    )
endforeach()

install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
  )

install(
  EXPORT "${targets_export_name}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
  )

