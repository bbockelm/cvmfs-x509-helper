#
# CMake build script to configure and build CernVM-FS' x509 authz helper.
#

cmake_minimum_required (VERSION 2.6.2)
set (PROJECT_NAME "CVMFS-X509-Helper")
project (${PROJECT_NAME})

message ("Running CMake version ${CMAKE_VERSION}")

#
# The version numbers
#
set (CVMFS-X509-Helper_VERSION_MAJOR 0)
set (CVMFS-X509-Helper_VERSION_MINOR 9)
set (CVMFS-X509-Helper_VERSION_PATCH 0)
set (CVMFS-X509-Helper_VERSION_STRING "${CVMFS-X509-Helper_VERSION_MAJOR}.${CVMFS-X509-Helper_VERSION_MINOR}.${CVMFS-X509-Helper_VERSION_PATCH}")

#
# set the path where cmake looks for additional modules
#
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

#
# detect the operating system and the distribution we are compiling on
#
if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set (MACOSX TRUE)
else (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set (MACOSX FALSE)
endif (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  set (LINUX TRUE)
  if (EXISTS /etc/debian_version)
    set (DEBIAN TRUE)
  endif (EXISTS /etc/debian_version)
  if (EXISTS /etc/arch-release)
    set (ARCHLINUX TRUE)
  endif (EXISTS /etc/arch-release)
else (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  set (LINUX FALSE)
  set (DEBIAN FALSE)
  set (ARCHLINUX FALSE)
endif (${CMAKE_SYSTEM_NAME} MATCHES "Linux")

#
# Set install prefix to /usr by default.
#
if (LINUX AND CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set (CMAKE_INSTALL_PREFIX "/usr" CACHE PATH "CVMFS install path default is /usr" FORCE)
  message("Setting default install prefix to ${CMAKE_INSTALL_PREFIX} on Linux")
endif (LINUX AND CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

#
# set default flags
#
# flags in CMAKE_C**_FLAGS are always passed to the compiler
#
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -fno-exceptions -fno-strict-aliasing -fasynchronous-unwind-tables -fno-omit-frame-pointer -fvisibility=hidden -Wall -D_REENTRANT -D__EXTENSIONS__ -D_LARGEFILE64_SOURCE -D__LARGE64_FILES")

set (INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORIES} src)

#
# check existence of include files
#
include (CheckIncludeFile)

macro (pedantic_include_check HEADERFILE VARIABLENAME)
  check_include_file (${HEADERFILE} ${VARIABLENAME})
  if (NOT ${VARIABLENAME})
    message (FATAL_ERROR "${HEADERFILE} is missing on your system")
  endif (NOT ${VARIABLENAME})
endmacro (pedantic_include_check)

macro (look_for_include_files)
  foreach (HEADER ${ARGV})
    string (REGEX REPLACE "/|\\." "_" HEADER_VAR_LOWER ${HEADER})
    string (TOUPPER "have_${HEADER_VAR_LOWER}" HEADER_VAR)
    pedantic_include_check (${HEADER} ${HEADER_VAR})
  endforeach (HEADER IN ITEMS ${ARGV})
endmacro (look_for_include_files)

set (REQUIRED_HEADERS alloca.h dlfcn.h errno.h fcntl.h inttypes.h limits.h
                      pthread.h  stdint.h sys/types.h syslog.h time.h unistd.h)
look_for_include_files (${REQUIRED_HEADERS})

set (INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORIES} ${CMAKE_BINARY_DIR})


# OpenSSL
find_package (OpenSSL REQUIRED)
set (INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORIES} ${OPENSSL_INCLUDE_DIR})

# VOMS / globus libraries for secure CVMFS
find_package (VOMS REQUIRED)
set (INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORIES} ${VOMS_INCLUDE_DIR})
add_definitions(-DVOMS_AUTHZ)

include (FindPkgConfig)
pkg_check_modules(GLOBUS REQUIRED globus-gsi-credential globus-gsi-cert-utils globus-common globus-gsi-sysconfig globus-gsi-callback)
set (INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORIES} ${GLOBUS_INCLUDE_DIRS})
add_definitions(-DGLOBUS_AUTHZ)

include_directories (${INCLUDE_DIRECTORIES} vjson)

add_subdirectory (vjson)
add_subdirectory (src)

#
# Documentation
#
#install (
#    FILES                   README AUTHORS ChangeLog COPYING
#    DESTINATION             share/doc/cvmfs-x509-helper-${CVMFS-X509-Helper_VERSION_STRING}
#    PERMISSIONS             OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ
#        )
