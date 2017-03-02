# Based on original work by David Manura
# Copyright (C) 2007-2012 LuaDist.
# Copyright (C) 2013 Brian Sidebotham

# Redistribution and use of this file is allowed according to the terms of the
# MIT license.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.


project( openssl )
cmake_minimum_required( VERSION 2.8.3 )

set(BUILD_OBJECT_LIBRARY_ONLY OFF)

set(SSLIncludeDir ${CMAKE_SOURCE_DIR}/include)
#set(Includes ${CMAKE_CURRENT_BINARY_DIR}/crypto ${CMAKE_CURRENT_BINARY_DIR}/ssl ${CMAKE_SOURCE_DIR}/crypto ${CMAKE_SOURCE_DIR}/.. ${CMAKE_SOURCE_DIR})
set(Includes ${SSLIncludeDir} ${CMAKE_CURRENT_BINARY_DIR}/ssl ${CMAKE_SOURCE_DIR}/crypto .)
message(STATUS "Include dirs: ${Includes}")
include_directories ( BEFORE SYSTEM ${Includes})

add_definitions( -DOPENSSL_NO_ASM )

if( WIN32 AND NOT CYGWIN )
  add_definitions( -DOPENSSL_SYSNAME_WIN32 )
  add_definitions( -DWIN32_LEAN_AND_MEAN )
endif ( )

if( MINGW )
  set( CMAKE_SHARED_LINKER_FLAGS "-Wl,--export-all" )
endif()

add_subdirectory( crypto )
add_subdirectory( ssl )

file( COPY e_os2.h e_os.h DESTINATION ${SSLIncludeDir}/openssl )

add_subdirectory( apps )

install( FILES tools/c_hash tools/c_info tools/c_issuer tools/c_name 
  #tools/c_rehash # This one is not contained in this branch for some reason, check here https://github.com/pol51/OpenSSL-CMake
  FAQ LICENSE PROBLEMS README README.ASN1 README.ENGINE
  DESTINATION share/openssl )

install( DIRECTORY doc DESTINATION ./ )

install( FILES e_os2.h DESTINATION include/openssl)

# Generate the package target
set( CPACK_GENERATOR ZIP TGZ )
set( CPACK_PACKAGE_NAME "openssl-cmake" )
set( CPACK_PACKAGE_VERSION_MAJOR 1  )
set( CPACK_PACKAGE_VERSION_MINOR 0  )
set( CPACK_PACKAGE_VERSION_PATCH 1u )

include( CPack )
