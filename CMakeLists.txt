CMAKE_MINIMUM_REQUIRED(VERSION 3.5 FATAL_ERROR)

# parse the full version number from jversion.h and include in JPEG_FULL_VERSION
FILE(READ jversion.h _jversion_h_contents)

STRING(REGEX REPLACE ".*#define[ \t]+JVERSION[ \t]+\"([0-9]+[a-z]+).+\".*" "\\1" JPEG_FULL_VERSION ${_jversion_h_contents})
STRING(REGEX REPLACE "([0-9]+)[a-z]+" "\\1" JPEG_MAJOR_VERSION ${JPEG_FULL_VERSION})
STRING(REGEX REPLACE "[0-9]+([a-z]+)" "\\1" JPEG_MINOR_VERSION ${JPEG_FULL_VERSION})

# Convert minor version letter to number
IF ("${JPEG_MINOR_VERSION}" MATCHES [aA])
  SET(JPEG_MINOR_VERSION 0)
ELSEIF("${JPEG_MINOR_VERSION}" MATCHES [bB])
  SET(JPEG_MINOR_VERSION 1)
ELSEIF("${JPEG_MINOR_VERSION}" MATCHES [cC])
  SET(JPEG_MINOR_VERSION 2)
ELSEIF("${JPEG_MINOR_VERSION}" MATCHES [dD])
  SET(JPEG_MINOR_VERSION 3)
ELSEIF("${JPEG_MINOR_VERSION}" MATCHES [eE])
  SET(JPEG_MINOR_VERSION 4)
ELSEIF("${JPEG_MINOR_VERSION}" MATCHES [fF])
  SET(JPEG_MINOR_VERSION 5)
ELSEIF("${JPEG_MINOR_VERSION}" MATCHES [gG])
  SET(JPEG_MINOR_VERSION 6)
ELSEIF("${JPEG_MINOR_VERSION}" MATCHES [hH])
  SET(JPEG_MINOR_VERSION 7)
ELSE()
  MESSAGE(FATAL_ERROR "Minor JPEG version exceeds version H")
ENDIF()

PROJECT(jpeg VERSION ${JPEG_MAJOR_VERSION}.${JPEG_MINOR_VERSION} LANGUAGES C)

# Use GNUInstallDirs to install libraries into correct
# locations on all platforms.
INCLUDE(GNUInstallDirs)

IF(MSVC)
  SET(jconfig "jconfig.vc")
ENDIF()

IF (MINGW)
  SET(jconfig "jconfig.vc")
ENDIF()

CONFIGURE_FILE(${jconfig} jconfig.h @ONLY)
CONFIGURE_FILE(jerror.h jerror.h COPYONLY)
CONFIGURE_FILE(jmorecfg.h jmorecfg.h COPYONLY)
CONFIGURE_FILE(jpeglib.h jpeglib.h COPYONLY)

SET(LIBJPEG_PUBLIC_HEADERS
  ${CMAKE_CURRENT_BINARY_DIR}/jconfig.h
  ${CMAKE_CURRENT_BINARY_DIR}/jerror.h
  ${CMAKE_CURRENT_BINARY_DIR}/jmorecfg.h
  ${CMAKE_CURRENT_BINARY_DIR}/jpeglib.h
)

SET(LIBJPEG_PRIVATE_HEADERS
  jdct.h
  jinclude.h
  jmemsys.h
  jpegint.h
  jversion.h
)

SET(LIBJPEG_SRCS
  jaricom.c
  jcapimin.c
  jcapistd.c
  jcarith.c
  jccoefct.c
  jccolor.c
  jcdctmgr.c
  jchuff.c
  jcinit.c
  jcmainct.c
  jcmarker.c
  jcmaster.c
  jcomapi.c
  jcparam.c
  jcprepct.c
  jcsample.c
  jctrans.c
  jdapimin.c
  jdapistd.c
  jdarith.c
  jdatadst.c
  jdatasrc.c
  jdcoefct.c
  jdcolor.c
  jddctmgr.c
  jdhuff.c
  jdinput.c
  jdmainct.c
  jdmarker.c
  jdmaster.c
  jdmerge.c
  jdpostct.c
  jdsample.c
  jdtrans.c
  jerror.c
  jfdctflt.c
  jfdctfst.c
  jfdctint.c
  jidctflt.c
  jidctfst.c
  jidctint.c
  jmemmgr.c
  jmemnobs.c
  jquant1.c
  jquant2.c
  jutils.c
)

# Dependent include files
INCLUDE_DIRECTORIES(${LIBJPEG_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

IF(MSVC)
  LIST(APPEND PRIVATE_DEFINITIONS -D_CRT_SECURE_NO_WARNINGS) # Disable warnings for unsecure CRT functions
  LIST(APPEND PRIVATE_DEFINITIONS -D_LIB)
  LIST(APPEND PRIVATE_OPTIONS /Gy) # Eliminate Duplicate Strings
  SET(CMAKE_DEBUG_POSTFIX "d")
ENDIF(MSVC)

ADD_LIBRARY(jpeg STATIC ${LIBJPEG_SRCS} ${LIBJPEG_PRIVATE_HEADERS} ${LIBJPEG_PUBLIC_HEADERS})

# Dependent include files
TARGET_INCLUDE_DIRECTORIES(jpeg PUBLIC 
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}> 
  $<INSTALL_INTERFACE:include>
  )

TARGET_COMPILE_DEFINITIONS(jpeg
  PRIVATE ${PRIVATE_DEFINITIONS}
  )
  
TARGET_COMPILE_OPTIONS(jpeg 
  PRIVATE ${PRIVATE_OPTIONS}
  )


# INSTALL
INSTALL(TARGETS jpeg EXPORT jpeg_export
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )

INSTALL(FILES 
  ${LIBJPEG_PUBLIC_HEADERS} 
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )
