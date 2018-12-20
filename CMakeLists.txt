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
cmake_minimum_required( VERSION 3.4 )

option( BUILD_OBJECT_LIBRARY_ONLY "Enable to only get OBJECT-type libraries, otherwise you get STATIC libraries" OFF )
option( OPENSSL_BUILD_APPS "Enable executable targets. Disable if you only want the libraries" ON )

set( SSLIncludeDir ${CMAKE_CURRENT_BINARY_DIR}/include )

set( definitions
    OPENSSL_NO_ASM
)

if( WIN32 AND NOT CYGWIN )
    list( APPEND definitions
        OPENSSL_SYSNAME_WIN32
        WIN32_LEAN_AND_MEAN
    )

    set( CMAKE_DEBUG_POSTFIX d )

endif()

if( MSVC )
    # Support parallel building of translation units with MSVC toolchain
    add_definitions(/MP)
else()
    SET(CMAKE_C_FLAGS "-fPIC")
endif()

if (MSVC)
    foreach(flag_var CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
        if(${flag_var} MATCHES "/MD")
          string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
        endif(${flag_var} MATCHES "/MD")
    endforeach(flag_var)
endif (MSVC AND OPENSSL_MSVC_STATIC_RUNTIME)

if( MINGW )
  set( CMAKE_SHARED_LINKER_FLAGS "-Wl,--export-all" )
endif()

file( COPY e_os2.h e_os.h DESTINATION ${SSLIncludeDir}/openssl )

add_subdirectory( crypto )
add_subdirectory( ssl )

if( OPENSSL_BUILD_APPS )
    add_subdirectory( apps )
endif()

install( FILES tools/c_hash tools/c_info tools/c_issuer tools/c_name
  #tools/c_rehash # This one is not contained in this branch for some reason, check here https://github.com/pol51/OpenSSL-CMake
  FAQ LICENSE PROBLEMS README README.ASN1 README.ENGINE
  DESTINATION share/openssl )

install( DIRECTORY doc/ DESTINATION doc )

install( FILES e_os2.h DESTINATION include/openssl )

# Generate the package target
set( CPACK_GENERATOR ZIP TGZ )
set( CPACK_PACKAGE_NAME "openssl-cmake" )
set( CPACK_PACKAGE_VERSION_MAJOR 1  )
set( CPACK_PACKAGE_VERSION_MINOR 0  )
set( CPACK_PACKAGE_VERSION_PATCH 1u )

include( CPack )
