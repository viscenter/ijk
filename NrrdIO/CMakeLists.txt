PROJECT(ITKNrrdIO)

INCLUDE_REGULAR_EXPRESSION("^.*.h$")
INCLUDE(TestBigEndian)

#---------------------------------------------------------

IF (NOT DEFINED ${IJK_DIR})
  GET_FILENAME_COMPONENT(IJK_ABSOLUTE_PATH "../.." ABSOLUTE)
  SET(IJK_DIR ${IJK_ABSOLUTE_PATH} CACHE PATH "IJK directory")
ENDIF (NOT DEFINED ${IJK_DIR})

SET(CMAKE_INSTALL_PREFIX "${IJK_DIR}/")

SET(QNANHIBIT 1)
SET(BIGBITFIELD 1)

SET(NRRDIO_DIR "src/NrrdIO")

#---------------------------------------------------------

#
# This CMake file configures the NrrdIO library build.  NrrdIO
# is used by Insight/Code/IO/itkNrrdIO for reading/writing 
# "Nearly Raw Raster Data" within the open-source Teem software 
# package. See http://teem.sourceforge.net for more information.
#

SET(nrrdio_SRCS comment.c enumsNrrd.c mop.c string.c 754.c defaultsNrrd.c
parseAir.c dio.c format.c parseNrrd.c formatEPS.c encoding.c
formatNRRD.c encodingAscii.c formatPNG.c encodingBzip2.c
formatPNM.c accessors.c encodingGzip.c formatText.c 
array.c encodingHex.c formatVTK.c read.c arraysNrrd.c encodingRaw.c 
gzio.c reorder.c write.c axis.c endianAir.c  keyvalue.c
biffbiff.c endianNrrd.c methodsNrrd.c  sane.c enum.c
miscAir.c simple.c )

# Turn on TEEM_BUILD so that the proper dll export def's are 
# used on windows builds.
ADD_DEFINITIONS(-DTEEM_BUILD=1)

# Set compiler flags for endian-ness.
#IF(CMAKE_WORDS_BIGENDIAN)
#  ADD_DEFINITIONS(-DTEEM_ENDIAN=4321)
#ELSE(CMAKE_WORDS_BIGENDIAN)
#  ADD_DEFINITIONS(-DTEEM_ENDIAN=1234)
#ENDIF(CMAKE_WORDS_BIGENDIAN)
TEST_BIG_ENDIAN(FLAG_BIGENDIAN)
IF(FLAG_BIGENDIAN)
  ADD_DEFINITIONS(-DTEEM_ENDIAN=4321)
ELSE(FLAG_BIGENDIAN)
  ADD_DEFINITIONS(-DTEEM_ENDIAN=1234)
ENDIF(FLAG_BIGENDIAN)

# Set compiler flags for 32 or 64 bit architecture (based on the size
# of a void pointer).
IF(CMAKE_SIZEOF_VOID_P MATCHES  8)
  ADD_DEFINITIONS(-DTEEM_32BIT=0)
ELSE(CMAKE_SIZEOF_VOID_P MATCHES 8)
  ADD_DEFINITIONS(-DTEEM_32BIT=1)
ENDIF(CMAKE_SIZEOF_VOID_P MATCHES 8)

# The BIGBITFIELD variable is configured by the root level CMakeLists.txt
IF(BIGBITFIELD)
  ADD_DEFINITIONS(-DTEEM_BIGBITFIELD=1)
ELSE(BIGBITFIELD)
  ADD_DEFINITIONS(-DTEEM_BIGBITFIELD=0)
ENDIF(BIGBITFIELD)

#The QNANHIBIT variable is configured by the root level CMakeLists.txt
IF(QNANHIBIT)
  ADD_DEFINITIONS(-DTEEM_QNANHIBIT=1)
ELSE(QNANHIBIT)
  ADD_DEFINITIONS(-DTEEM_QNANHIBIT=0)
ENDIF(QNANHIBIT)

#DirectIO is the fast way to do multi-gigabyte I/O and currently only available
#for SGI platforms.  Use of DirectIO is enabled manually for now.
#OPTION(USE_DIRECTIO "Use DirectIO for Nrrd file IO.  Only valid on SGI systems." 0)
#MARK_AS_ADVANCED(USE_DIRECTIO)
#IF(USE_DIRECTIO)
#  ADD_DEFINITIONS(-DTEEM_DIO=1)
#ELSE(USE_DIRECTIO)
  ADD_DEFINITIONS(-DTEEM_DIO=0)
#ENDIF(USE_DIRECTIO)

# turn on compression library
# Note: requires zlib 
# (i.e., programs compiled with ITKNrrd lib must also be compiled with zlib)
ADD_DEFINITIONS(-DTEEM_ZLIB=1)

ADD_LIBRARY(ITKNrrdIO ${nrrdio_SRCS} )
TARGET_LINK_LIBRARIES(ITKNrrdIO ${ITK_ZLIB_LIBRARIES} )

#INSTALL_TARGETS(/lib/InsightToolkit ITKNrrdIO)
#INSTALL_FILES(/include/InsightToolkit/Utilities/NrrdIO "(\\.h)$")

#################################################
# IJK installation instructions
#################################################

IF (NOT CMAKE_BUILD_TYPE)
  SET (CMAKE_BUILD_TYPE Release CACHE STRING 
       "Default build type: Release" FORCE)
ENDIF (NOT CMAKE_BUILD_TYPE)

INSTALL(TARGETS ITKNrrdIO DESTINATION lib)
INSTALL(FILES "NrrdIO.h" DESTINATION "include")

#---------------------------------------------------------

ADD_CUSTOM_TARGET(tar WORKING_DIRECTORY ../.. COMMAND tar cvf ${NRRDIO_DIR}/NrrdIO.tar ${NRRDIO_DIR}/README ${NRRDIO_DIR}/INSTALL ${NRRDIO_DIR}/*.c ${NRRDIO_DIR}/*.h ${NRRDIO_DIR}/CMakeLists.txt )
