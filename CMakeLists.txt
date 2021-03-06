############################################
#
# oenixCore CMake
#
############################################

CMAKE_MINIMUM_REQUIRED( VERSION 2.8 )

PROJECT( Bea )


############################################
# Disallow in-source builds
############################################

IF( "${Bea_SOURCE_DIR}" STREQUAL "${Bea_BINARY_DIR}" )
	MESSAGE( FATAL_ERROR "\nBea likes to be built out-of-source.\n"
			"Please create a separate build directory and\n"
			"run cmake there ( cmake .. )" )
ENDIF ( "${Bea_SOURCE_DIR}" STREQUAL "${Bea_BINARY_DIR}" )

############################################
# Check Dependencies
############################################

MESSAGE ( "-- Checking Dependencies: " )

FIND_PACKAGE( Threads REQUIRED )
SET( LIBRARIES ${LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} )

set(Boost_USE_STATIC_LIBS   ON)
set(Boost_USE_MULTITHREADED  ON)
FIND_PACKAGE( Boost 1.39 REQUIRED COMPONENTS date_time thread )
IF( Boost_FOUND )
    MESSAGE( " Found Boost" )
else( Boost_FOUND )
    MESSAGE( FATAL_ERROR " Boost Not Found!" )
endif( Boost_FOUND )
include_directories( ${Boost_INCLUDE_DIRS} )
SET( LIBRARIES ${LIBRARIES} ${Boost_LIBRARIES} )
link_directories ( ${Boost_LIBRARY_DIRS} )

FIND_PACKAGE( OpenGL REQUIRED )
if( OPENGL_FOUND )
    MESSAGE( " Found OpenGL" )
else(OPENGL_FOUND)
    MESSAGE( FATAL_ERROR " OpenGL Not Found!" )
endif(OPENGL_FOUND)
SET( INCLUDES ${INCLUDES} ${OPENGL_INCLUDE_DIR} )
MESSAGE(" OGL Libraries: " ${OPENGL_LIBRARIES} )
SET( LIBRARIES ${LIBRARIES} ${OPENGL_LIBRARIES} )

# FIND_PACKAGE( GLU REQUIRED )
# if( GLU_FOUND )
#     MESSAGE( " Found Glu" )
# else(GLU_FOUND)
#     MESSAGE( FATAL_ERROR " GLu Not Found!" )
# endif(GLU_FOUND)
# SET( INCLUDES ${INCLUDES} ${GLU_INCLUDE_DIR} )
# SET( LIBRARIES ${LIBRARIES} ${GLU_LIBRARIES} )


#MESSAGE(" Libraries: " ${LIBRARIES} )

############################################
# Find GLFW
############################################

set( SEARCH_PATHS
    ${GLFW_PATH}
    ${GLFW_PATH}/include
    ${GLFW_PATH}/lib
    ${GLFW_PATH}/lib/win32/Release # 2.7-lite
    ${GLFW_PATH}/lib/win32/Debug
    ${GLFW_PATH}/support/msvc80/Release # 2.6 Window Support
    ${GLFW_PATH}/support/msvc80/Debug
    )

FIND_PATH( GLFW_INCLUDE_DIR GL/glfw.h PATHS ${SEARCH_PATHS} )
FIND_LIBRARY( GLFW_LIBRARY glfw PATHS ${SEARCH_PATHS} )

IF( GLFW_INCLUDE_DIR AND GLFW_LIBRARY )
    SET( GLFW_FOUND TRUE )
ENDIF( GLFW_INCLUDE_DIR AND GLFW_LIBRARY )

IF( GLFW_FOUND )
    MESSAGE (" Found GLFW ")
ELSE( GLFW_FOUND )
    MESSAGE( FATAL_ERROR "Could not find GLFW! Please provide the path \n"
                        "containing the 'lib' and 'include' subdirectories \n"
                        "that have the glfw header and static library via \n"
                        "-DGLFW_PATH=\"path\"" )
ENDIF( GLFW_FOUND )

SET( INCLUDES ${INCLUDES} ${GLFW_INCLUDE_DIR} )
SET( LIBRARIES ${LIBRARIES} ${GLFW_LIBRARY} )

## GLFW Window Move Hack.

if( _GLFW_WM_MOVE_HACK )
    set_property( DIRECTORY
    PROPERTY COMPILE_DEFINITIONS _GLFW_WM_MOVE_HACK
    )
endif()

#3


############################################
# Box2d
############################################

MESSAGE( "- Finding Box2D" )

set( SEARCH_PATHS 
${BOX2D_PATH}
${BOX2D_PATH}/Include
${BOX2D_PATH}/Library
)

FIND_PATH( BOX2D_INCLUDE_DIR Box2D.h PATHS ${SEARCH_PATHS} )
FIND_LIBRARY( BOX2D_LIBRARY Box2D PATHS ${SEARCH_PATHS} )

if( BOX2D_INCLUDE_DIR AND BOX2D_LIBRARY )
    SET( BOX2D_FOUND TRUE )
endif()

IF( BOX2D_FOUND )
    MESSAGE (" Found Box2d ")
ELSE( BOX2D_FOUND )
    MESSAGE( FATAL_ERROR "Could not find Box2d! Please provide the path \n"
					    "containing the 'Library' and 'Include' subdirectories \n"
					    "that have the Box2d header and static library via \n"
					    "-DBOX2D_PATH=\"path\"" )
ENDIF(  BOX2D_FOUND )

SET( INCLUDES ${INCLUDES} ${BOX2D_INCLUDE_DIR} )
SET( LIBRARIES ${LIBRARIES} ${BOX2D_LIBRARY} )
    

############################################
# Platform Stuff
############################################

#WINDOWS (MSVC)
if(WIN32)
	
	if(MSVC)
	    MESSAGE( "Building for MSVC\n" )
        ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS) # No stupid warning about fopen and sprintf
    endif()

	ADD_DEFINITIONS(-DWIN32 -DMSVC)
	

endif(WIN32)


# Linux
if(UNIX)

    MESSAGE( "-- Finding Unix Specific Libraries" )

    FIND_LIBRARY( XRANDR_LIBRARY Xrandr )
    IF( XRANDR_LIBRARY )
        MESSAGE(" Found Xrandr ")
    ELSE( XRANDR_LIBRARY)
        MESSAGE( FATAL_ERROR " Could not fix XRandR! \n"
                "GLFW requires XRandR on linux.\n" )
    ENDIF( XRANDR_LIBRARY )

    SET( LIBRARIES ${LIBRARIES} ${XRANDR_LIBRARY} )

    set(CMAKE_CXX_FLAGS "-std=c++0x")

endif(UNIX )


############################################
# Find PhoenixCore
############################################

set( SEARCH_PATHS
    ${PHOENIX_PATH}/source
    ${PHOENIX_PATH}/build/lib
    ${PHOENIX_PATH}/build/lib/Release
    ${PHOENIX_PATH}/build/lib/Debug
    )
    
FIND_PATH( PHOENIX_INCLUDE_DIR Phoenix.h PATHS ${SEARCH_PATHS} )
FIND_LIBRARY( PHOENIX_LIBRARY PhoenixCore PATHS ${SEARCH_PATHS} )

IF( PHOENIX_INCLUDE_DIR AND PHOENIX_LIBRARY )
    SET( PHOENIX_FOUND TRUE )
ENDIF( )

IF( PHOENIX_FOUND )
    MESSAGE (" Found PhoenixCore ")
ELSE( PHOENIX_FOUND )
    MESSAGE( FATAL_ERROR "Could not find Phoenix! Please provide the path \n"
                        "containing the 'lib' and 'include' subdirectories \n"
                        "that have the phoenixcore header and static library via \n"
                        "-DPHOENIX_PATH=\"path\"" )
ENDIF( PHOENIX_FOUND )

SET( INCLUDES ${INCLUDES} ${PHOENIX_INCLUDE_DIR} )
SET( LIBRARIES ${PHOENIX_LIBRARY} ${LIBRARIES})

############################################
# Everything looks good.
############################################

############################################
# Change the output directories
############################################
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_OUTPUT_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

############################################
# Include Directories
############################################

include_directories( ${Bea_SOURCE_DIR}/source ${INCLUDES})
include_directories( ${Bea_SOURCE_DIR}/source/tmx-parser ${INCLUDES})
include_directories( ${Bea_SOURCE_DIR}/source/tmx-parser/tinyxml ${INCLUDES})
include_directories( ${Bea_SOURCE_DIR}/test-source ${INCLUDES})

############################################
# Set the path for the data dir
############################################

ADD_DEFINITIONS( -DDATA_DIR="${Bea_SOURCE_DIR}/data/" )

############################################
# Include the source CMakeLists.txt
############################################

subdirs( source )
subdirs( test-source )

MESSAGE(" Libraries: " ${LIBARIES} )