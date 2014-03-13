cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME box2d)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# BOX2D
#------------------------------------------------------------------------------
# Google test as an external project is a bad idea! You really should add it
# as a subdirectory so that it can capture your compiler flags
set(BOX2D_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/box2d)
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${BOX2D_DIR}
  SVN_REPOSITORY "http://box2d.googlecode.com/svn/trunk"
  SVN_REVISION "278"
  USE_CACHING TRUE
  )

set(BOX2D_INSTALL OFF CACHE BOOL "Install box2d")
set(BOX2D_BUILD_EXAMPLES OFF CACHE BOOL "Build box2d examples (requires OpenGL).")
set(BOX2D_BUILD_STATIC ON CACHE BOOL "Build static box2d library.")

# Add gtest now that we have the appropriate flags set.
add_subdirectory(${BOX2D_DIR}/Box2D)

CPM_ExportAdditionalIncludeDir("${BOX2D_DIR}/Box2D")
CPM_ExportAdditionalLibraryTarget("Box2D")

