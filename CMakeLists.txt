cmake_minimum_required (VERSION 3.1.3)

# Before the Project!
SET(CMAKE_CXX_STANDARD 11 ) 			# We want at least C++11 Standard
SET(CMAKE_CXX_STANDARD_REQUIRED On) 	# .... and it is required

SET(CMAKE_VERBOSE_MAKEFILE ON)
SET(CMAKE_COLOR_MAKEFILE ON)
SET(CMAKE_ERROR_DEPRECATED ON)
SET(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Project Name + Languages - C is here because we require pthreads for testing
project (FastLED CXX C)
# Project description - only used in doxygen and cpack
set(PROJECT_DESCRIPTION "FastLED library see: http://fastled.io")

# Some arbitrary hand-maintained version
set(PROJECT_MAJOR_VERSION 3)
set(PROJECT_MINOR_VERSION 1)
set(PROJECT_PATCH_VERSION 0)
set(PROJECT_VERSION  ${PROJECT_MAJOR_VERSION}.${PROJECT_MINOR_VERSION}.${PROJECT_PATCH_VERSION})

# User did not provide install prefix via cmake commandline
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT )
  SET(CMAKE_INSTALL_PREFIX ${STAGING_DIR} )
endif(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

#-------------------------------------------------------------------------------
# Print Summary
#-------------------------------------------------------------------------------
message(STATUS "------------- Config ${PROJECT_NAME} -------------" )
message(STATUS " Build-Type:\t\t ${CMAKE_BUILD_TYPE} " )
message(STATUS " CPU:\t\t ${ARDUINO_MODEL} ")
message(STATUS " Install Prefix:\t ${CMAKE_INSTALL_PREFIX} ")
message(STATUS "" )

#-------------------------------------------------------------------------------
# Input configuration
#-------------------------------------------------------------------------------
# This project has no include / src directory separation
SET(PROJECT_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${PROJECT_INCLUDE_DIR})

# add . to the includes during the build - redundant here
set(CMAKE_INCLUDE_CURRENT_DIR ON)

#------------------------------
# Import Arduino base
#------------------------------
find_package(Arduino COMPONENTS core CONFIG REQUIRED)

#-----------
# sources
#-----------
set(SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/power_mgt.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/noise.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/lib8tion.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/hsv2rgb.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/FastLED.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/colorutils.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/colorpalettes.cpp
)

#-------------------------------------------------------------------------------
# Output  configuration
#-------------------------------------------------------------------------------

# projectname = name of library (without lib- prefix)
string(TOLOWER "${PROJECT_NAME}" LIBRARY_NAME)
#Since the "standard" cmake template works with component name set it
set(COMPONENT_NAME ${PROJECT_NAME})

# Make debug libs visible per default ->  SET_TARGET_PROPERTIES
SET(CMAKE_DEBUG_POSTFIX "_dbg")
SET(CMAKE_TESTCOVERAGE_POSTFIX "_cov")
# (CMAKE_BINARY_DIR = where cmake was invoked and found project())
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY	"${PROJECT_BINARY_DIR}/bin/")
# Folder for libraries (during build)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY	"${PROJECT_BINARY_DIR}/lib/")
# Put Archives + Libraries togther
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})


# Offer the user the choice of overriding the installation directories
set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDE_DIR include CACHE PATH  "Installation directory for header files")
set(INSTALL_CMAKE_DIR ${INSTALL_LIB_DIR}/cmake/ CACHE PATH
    "Installation directory for CMake files")
set(INSTALL_DOC_DIR "doc/${PROJECT_NAME}" CACHE PATH  "Installation directory for doxygen docs")

export(PACKAGE ${PROJECT_NAME})
# Configuration
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

#------------------------------
# Output a library
#------------------------------
ADD_LIBRARY(${LIBRARY_NAME}  ${SRCS} )

SET_TARGET_PROPERTIES(${LIBRARY_NAME} PROPERTIES
	VERSION ${LIBRARY_VERSION}
	SOVERSION ${LIBRARY_VERSION}
  )


target_include_directories(
  ${LIBRARY_NAME}
  PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  )

# Add dependencies to library to pass
# dependencies on to client executable
TARGET_LINK_LIBRARIES( ${LIBRARY_NAME}
  Arduino::core
  )

#-----
# Install Targets Library + Headers
#-----
MESSAGE(STATUS "** Generating Package Configurations **")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${version_config}
  VERSION ${LIBRARY_VERSION}
  COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Note: variable 'targets_export_name' used
configure_file("${CMAKE_SOURCE_DIR}/cmake/Config.cmake.in"
  "${component_config}" @ONLY)

INSTALL(TARGETS ${LIBRARY_NAME}
  EXPORT ${targets_export_name}
  COMPONENT DEVELOP
  ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
  LIBRARY DESTINATION ${INSTALL_LIB_DIR}
  # this will only add -I<prefix>/include/fastled to client compile flags
  INCLUDES DESTINATION ${INSTALL_INCLUDE_DIR}/${LIBRARY_NAME}
  )


# --------------------------------------------------------------------
# Install public headers
# --------------------------------------------------------------------

INSTALL(DIRECTORY
   ${CMAKE_CURRENT_SOURCE_DIR}/platforms
   COMPONENT DEVELOP
   DESTINATION ${INSTALL_INCLUDE_DIR}/${LIBRARY_NAME}
   FILES_MATCHING PATTERN "*.h"
   )

INSTALL(DIRECTORY
   ${CMAKE_CURRENT_SOURCE_DIR}/lib8tion
   COMPONENT DEVELOP
   DESTINATION ${INSTALL_INCLUDE_DIR}/${LIBRARY_NAME}
   FILES_MATCHING PATTERN "*.h"
   )

 INSTALL(FILES
   ${CMAKE_CURRENT_SOURCE_DIR}/bitswap.h
   ${CMAKE_CURRENT_SOURCE_DIR}/colorpalettes.h
   ${CMAKE_CURRENT_SOURCE_DIR}/cpp_compat.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastled_delay.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastpin.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastspi.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastspi_types.h
   ${CMAKE_CURRENT_SOURCE_DIR}/lib8tion.h
   ${CMAKE_CURRENT_SOURCE_DIR}/pixeltypes.h
   ${CMAKE_CURRENT_SOURCE_DIR}/chipsets.h
   ${CMAKE_CURRENT_SOURCE_DIR}/colorutils.h
   ${CMAKE_CURRENT_SOURCE_DIR}/dmx.h
   ${CMAKE_CURRENT_SOURCE_DIR}/FastLED.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastspi_bitbang.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastspi_nop.h
   ${CMAKE_CURRENT_SOURCE_DIR}/hsv2rgb.h
   ${CMAKE_CURRENT_SOURCE_DIR}/noise.h
   ${CMAKE_CURRENT_SOURCE_DIR}/platforms.h
   ${CMAKE_CURRENT_SOURCE_DIR}/color.h
   ${CMAKE_CURRENT_SOURCE_DIR}/controller.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastled_config.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastled_progmem.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastspi_dma.h
   ${CMAKE_CURRENT_SOURCE_DIR}/fastspi_ref.h
   ${CMAKE_CURRENT_SOURCE_DIR}/led_sysdefs.h
   ${CMAKE_CURRENT_SOURCE_DIR}/pixelset.h
   ${CMAKE_CURRENT_SOURCE_DIR}/power_mgt.h
   COMPONENT DEVELOP
   DESTINATION ${INSTALL_INCLUDE_DIR}/${LIBRARY_NAME}
   )

# --------------------------------------------------------------------
# Cmake Configuration file infrastructure
# --------------------------------------------------------------------
MESSAGE(STATUS "** Generating Package Configurations **")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${version_config}
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Note: variable 'targets_export_name' used
configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)


# Version config file
INSTALL(
    FILES "${project_config}" "${version_config}"
    COMPONENT DEVELOP
    DESTINATION "${INSTALL_CMAKE_DIR}/${COMPONENT_NAME}"
)

# Config
install(
    EXPORT "${targets_export_name}"
    COMPONENT DEVELOP
    NAMESPACE "${namespace}"
    DESTINATION "${INSTALL_CMAKE_DIR}/${COMPONENT_NAME}"
)

# --------------------------------------------------------------------
# Generate CPACK installer
# --------------------------------------------------------------------
set(CPACK_GENERATOR TGZ)
set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR "FastLED.io")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${PROJECT_DESCRIPTION})
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_INSTALL_DIRECTORY ${PROJECT_NAME}-${PROJECT_VERSION})
SET(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_COMPONENT_DEVELOP_DISPLAY_NAME "Libraries + Headers")
set(CPACK_COMPONENT_APIDOC_DISPLAY_NAME "API Documentation")

set(CPACK_COMPONENT_DEVELOP_DESCRIPTION "Components needed for development")
set(CPACK_COMPONENT_APIDOC_DESCRIPTION  "API Documentation")

SET(CPACK_NSIS_CONTACT "t.ruschival@gmail.com")
SET(CPACK_NSIS_MODIFY_PATH ON)

INCLUDE(CPack)

MESSAGE(STATUS "** end CMakeLists.txt **\n")
