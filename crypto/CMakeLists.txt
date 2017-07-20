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


include( CMakeParseArguments )

file( READ opensslconf.h.in CONF )
string( REPLACE "#undef OPENSSL_EXPORT_VAR_AS_FUNCTION" "#cmakedefine OPENSSL_EXPORT_VAR_AS_FUNCTION"
    CONF "${CONF}" )

set( CONF "
#define OPENSSL_NO_GMP
#define OPENSSL_NO_JPAKE
#define OPENSSL_NO_KRB5
#define OPENSSL_NO_MD2
#define OPENSSL_NO_RFC3779
#define OPENSSL_NO_STORE
#define OPENSSL_NO_DYNAMIC_ENGINE
#define OPENSSL_NO_SCTP
#define OPENSSL_NO_EC_NISTP_64_GCC_128
${CONF}" )
file( WRITE ${CMAKE_CURRENT_BINARY_DIR}/opensslconf.h.cmake "${CONF}" )

#~2DO: set BUILDINF_DATE to `date LC_ALL=C LC_TIME=C`
configure_file( buildinf.h.cmake buildinf.h )
include_directories(${CMAKE_CURRENT_BINARY_DIR})

if( WIN32 AND MINGW AND NOT CYGWIN )
  set( OPENSSL_EXPORT_VAR_AS_FUNCTION true )
endif()
configure_file( ${CMAKE_CURRENT_BINARY_DIR}/opensslconf.h.cmake ${SSLIncludeDir}/openssl/opensslconf.h )

include_directories( BEFORE SYSTEM asn1 evp modes )

if( BUILD_SHARED_LIBS )
  list( APPEND definitions OPENSSL_BUILD_SHLIBCRYPTO )
endif()

# avoid conflict: ocsp.h and wincrypt.h
if( WIN32 AND NOT CYGWIN )
  set_property( SOURCE ../engines/e_capi.c APPEND PROPERTY COMPILE_DEFINITIONS HEADER_OCSP_H )
  #set_property( SOURCE ../engines/ccgost/gost_asn1.c APPEND PROPERTY COMPILE_DEFINITIONS
  # WIN32_LEAN_AND_MEAN )
endif()

include( source_files.cmake )

file( COPY ${LIBSRC} DESTINATION ${SSLIncludeDir}/openssl FILES_MATCHING REGEX "\.h$" )

if(BUILD_OBJECT_LIBRARY_ONLY)
    list( APPEND definitions OPENSSL_THREADS )

    if( WIN32 AND MINGW AND NOT CYGWIN )
        list( APPEND definitions DSO_WIN32 WIN32_LEAN_AND_MEAN )
    endif()

    add_library( crypto OBJECT ${LIBSRC} ${OBJECTS_SRC} )
else()
    add_library( crypto ${LIBSRC} ${OBJECTS_SRC} )
    target_include_directories( crypto
        PUBLIC ${SSLIncludeDir} . ..
    )
    target_compile_definitions( crypto PUBLIC ${definitions} )

    if( WIN32 AND NOT CYGWIN )
        target_link_libraries( crypto PUBLIC ws2_32 crypt32 )
    endif()

    install( DIRECTORY include/ DESTINATION include/openssl )

    install( TARGETS crypto
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
    )
endif()
