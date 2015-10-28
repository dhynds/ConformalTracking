########################################################
# cmake file for building ConformalTracking
# - based on $MARLIN/example/mymarlin/CMakeLists.txt
# @author Daniel Hynds CERN-PH-LCD
CMAKE_MINIMUM_REQUIRED(VERSION 2.6 FATAL_ERROR)
########################################################


# project name
PROJECT( ConformalTracking )


# project version
SET( ${PROJECT_NAME}_VERSION_MAJOR 1 )
SET( ${PROJECT_NAME}_VERSION_MINOR 0 )
SET( ${PROJECT_NAME}_VERSION_PATCH 0 )



### DEPENDENCIES ############################################################

FIND_PACKAGE( ILCUTIL REQUIRED COMPONENTS ILCSOFT_CMAKE_MODULES )

# load default settings from ILCSOFT_CMAKE_MODULES
INCLUDE( ilcsoft_default_settings )


FIND_PACKAGE( ROOT 5.27 REQUIRED COMPONENTS MathMore Minuit )
LINK_LIBRARIES( ${ROOT_COMPONENT_LIBRARIES} ${ROOT_LIBRARIES} )

FIND_PACKAGE( MarlinUtil 1.4 REQUIRED )
FIND_PACKAGE( GSL REQUIRED )

FIND_PACKAGE( Marlin 1.0 REQUIRED ) # minimum required Marlin version
INCLUDE_DIRECTORIES( ${Marlin_INCLUDE_DIRS} )
LINK_LIBRARIES( ${Marlin_LIBRARIES} )
ADD_DEFINITIONS( ${Marlin_DEFINITIONS} )

FIND_PACKAGE( MarlinTrk REQUIRED )
INCLUDE_DIRECTORIES( ${MarlinTrk_INCLUDE_DIRS} )
LINK_LIBRARIES( ${MarlinTrk_LIBRARIES} )
ADD_DEFINITIONS( ${MarlinTrk_DEFINITIONS} )

FIND_PACKAGE( AIDA REQUIRED )
INCLUDE_DIRECTORIES( ${AIDA_INCLUDE_DIRS} )
LINK_LIBRARIES( ${AIDA_LIBRARIES} )
ADD_DEFINITIONS( ${AIDA_DEFINITIONS} )

find_package( DD4hep REQUIRED)
set(CMAKE_MODULE_PATH  ${CMAKE_MODULE_PATH}  ${DD4hep_ROOT}/cmake )
include( DD4hep )

if(DD4HEP_USE_XERCESC)
	find_package(XercesC)
endif()
include(DD4hep_XML_setup)
INCLUDE_DIRECTORIES( ${DD4hep_INCLUDE_DIRS} )
LINK_LIBRARIES( ${DD4hep_LIBRARIES} )

OPTION( MARLINRECO_AIDA "Set to ON to build MarlinReco with AIDA" ON )
IF( MARLINRECO_AIDA )
	FIND_PACKAGE( AIDA )
	IF( NOT AIDA_FOUND )
		MESSAGE( SEND_ERROR "AIDA not found. Please set MARLINRECO_AIDA to OFF or AIDA_DIR=/path/to/AIDA" )
	ENDIF()
ENDIF()

# optional package
FOREACH( pkg Marlin MarlinUtil GSL AIDA ROOT )
	IF( ${pkg}_FOUND )
		INCLUDE_DIRECTORIES( ${${pkg}_INCLUDE_DIRS} )
		LINK_LIBRARIES( ${${pkg}_LIBRARIES} )
		ADD_DEFINITIONS( ${${pkg}_DEFINITIONS} )
		MESSAGE( STATUS "${pkg} -- found" )
	ELSE()
    MESSAGE( STATUS "${pkg} -- not found" )
	ENDIF()
ENDFOREACH()



### DOCUMENTATION ###########################################################

OPTION( INSTALL_DOC "Set to OFF to skip build/install Documentation" OFF )

IF( INSTALL_DOC AND EXISTS "${PROJECT_SOURCE_DIR}/doc/CMakeLists.txt" )
    ADD_SUBDIRECTORY( ./doc )
ENDIF()



### LIBRARY #################################################################

# definitions to pass to the compiler
ADD_DEFINITIONS( "-Wall -ansi -pedantic" )
ADD_DEFINITIONS( "-Wno-long-long" )

# include directories
INCLUDE_DIRECTORIES( ./include )
#INSTALL_DIRECTORY( ./include DESTINATION . FILES_MATCHING PATTERN "*.h" )

# add library
AUX_SOURCE_DIRECTORY( ./src library_sources )
ADD_SHARED_LIBRARY( ${PROJECT_NAME} ${library_sources} )
INSTALL_SHARED_LIBRARY( ${PROJECT_NAME} DESTINATION lib )

# display some variables and write them to cache
DISPLAY_STD_VARIABLES()
