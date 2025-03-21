cmake_minimum_required(VERSION 3.15)
project(genicam CXX)

# Include GNUInstallDirs to provide standard installation directory variables
include(GNUInstallDirs)

# Architecture detection
if (UNIX)
  # Try to get architecture from compiler
  EXECUTE_PROCESS(
    COMMAND ${CMAKE_CXX_COMPILER} -dumpmachine
    COMMAND tr -d "\n"
    OUTPUT_VARIABLE CXX_MACHINE
  )
  string(REGEX REPLACE "([a-zA-Z_0-9]+).*" "\\1" ARCHITECTURE ${CXX_MACHINE})
elseif (WIN32)
  if ("${CMAKE_SIZEOF_VOID_P}" STREQUAL "4")
    set(ARCHITECTURE WIN32_i86)
  else ()
    set(ARCHITECTURE WIN64_x64)
  endif ()
endif ()

message(STATUS "Detected architecture ${ARCHITECTURE}")

# Setup library names, directories, and suffix based on architecture
set(GENICAM_LIB_SUFFIX)
if ("${ARCHITECTURE}" STREQUAL "arm")
  set(GENICAM_LIBRARIES
    libGCBase_gcc494_v3_4
    libGenApi_gcc494_v3_4
    liblog4cpp_gcc494_v3_4
    libLog_gcc494_v3_4
    libMathParser_gcc494_v3_4
    libNodeMapData_gcc494_v3_4
    libXmlParser_gcc494_v3_4
  )
  set(GENICAM_LIBRARIES_DIR ${CMAKE_CURRENT_SOURCE_DIR}/bin/Linux32_ARMhf)
  set(GENICAM_LIB_SUFFIX ".so")
elseif ("${ARCHITECTURE}" STREQUAL "aarch64")
  set(GENICAM_LIBRARIES
    libGCBase_gcc49_v3_4
    libGenApi_gcc49_v3_4
    liblog4cpp_gcc49_v3_4
    libLog_gcc49_v3_4
    libMathParser_gcc49_v3_4
    libNodeMapData_gcc49_v3_4
    libXmlParser_gcc49_v3_4
  )
  set(GENICAM_LIBRARIES_DIR ${CMAKE_CURRENT_SOURCE_DIR}/bin/Linux64_ARM)
  set(GENICAM_LIB_SUFFIX ".so")
elseif ("${ARCHITECTURE}" STREQUAL "i686")
  set(GENICAM_LIBRARIES
    libGCBase_gcc48_v3_4
    libGenApi_gcc48_v3_4
    liblog4cpp_gcc48_v3_4
    libLog_gcc48_v3_4
    libMathParser_gcc48_v3_4
    libNodeMapData_gcc48_v3_4
    libXmlParser_gcc48_v3_4
  )
  set(GENICAM_LIBRARIES_DIR ${CMAKE_CURRENT_SOURCE_DIR}/bin/Linux32_i86)
  set(GENICAM_LIB_SUFFIX ".so")
elseif ("${ARCHITECTURE}" STREQUAL "x86_64")
  set(GENICAM_LIBRARIES
    libGCBase_gcc48_v3_4
    libGenApi_gcc48_v3_4
    liblog4cpp_gcc48_v3_4
    libLog_gcc48_v3_4
    libMathParser_gcc48_v3_4
    libNodeMapData_gcc48_v3_4
    libXmlParser_gcc48_v3_4
  )
  set(GENICAM_LIBRARIES_DIR ${CMAKE_CURRENT_SOURCE_DIR}/bin/Linux64_x64)
  set(GENICAM_LIB_SUFFIX ".so")
elseif ("${ARCHITECTURE}" STREQUAL "WIN32_i86")
  set(GENICAM_LIBRARIES
    GCBase_MD_VC141_v3_4
    GenApi_MD_VC141_v3_4
  )
  set(GENICAM_LIBRARIES_DIR ${CMAKE_CURRENT_SOURCE_DIR}/library/CPP/lib/Win32_i86)
  set(GENICAM_LIB_SUFFIX ".lib")
elseif ("${ARCHITECTURE}" STREQUAL "WIN64_x64")
  set(GENICAM_LIBRARIES
    GCBase_MD_VC141_v3_4
    GenApi_MD_VC141_v3_4
  )
  set(GENICAM_LIBRARIES_DIR ${CMAKE_CURRENT_SOURCE_DIR}/library/CPP/lib/Win64_x64)
  set(GENICAM_LIB_SUFFIX ".lib")
else ()
  message(FATAL_ERROR "Unknown architecture")
endif ()

# Build the full paths for each library file
set(GENICAM_LIBRARIES_IN_BUILD_DIR)
foreach(GENICAM_LIB ${GENICAM_LIBRARIES})
  set(GENICAM_LIB_FILE "${GENICAM_LIBRARIES_DIR}/${GENICAM_LIB}${GENICAM_LIB_SUFFIX}")
  list(APPEND GENICAM_LIBRARIES_IN_BUILD_DIR "${GENICAM_LIB_FILE}")
endforeach()

# Create an INTERFACE target to expose headers and library paths
add_library(genicam INTERFACE)
target_include_directories(genicam INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/library/CPP/include)
target_link_libraries(genicam INTERFACE ${GENICAM_LIBRARIES_IN_BUILD_DIR})

# Only run install commands when repo A is the main project
if (CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
  # For Windows architectures, install binary directories (if applicable)
  if ("${ARCHITECTURE}" STREQUAL "WIN32_i86" OR "${ARCHITECTURE}" STREQUAL "WIN64_x64")
    install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/${ARCHITECTURE}/
      COMPONENT bin
      DESTINATION ${CMAKE_INSTALL_BINDIR}
    )
  endif()

  install(FILES ${GENICAM_LIBRARIES_IN_BUILD_DIR}
    COMPONENT bin
    DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )

  # Install headers only if shared libraries are being built
  if (BUILD_SHARED_LIBS)
    install(DIRECTORY library/CPP/include/
      COMPONENT dev
      DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )
  endif()
endif()
