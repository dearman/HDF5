cmake_minimum_required (VERSION 3.2.2)
PROJECT (HDF5_TOOLS_H5DIFF)

#-----------------------------------------------------------------------------
# Setup include Directories
#-----------------------------------------------------------------------------
INCLUDE_DIRECTORIES (${HDF5_TOOLS_SRC_DIR}/lib)

# --------------------------------------------------------------------
# Add the h5diff executables
# --------------------------------------------------------------------
add_executable (h5diff
    ${HDF5_TOOLS_H5DIFF_SOURCE_DIR}/h5diff_common.c
    ${HDF5_TOOLS_H5DIFF_SOURCE_DIR}/h5diff_main.c
)
TARGET_NAMING (h5diff STATIC)
TARGET_C_PROPERTIES (h5diff STATIC " " " ")
target_link_libraries (h5diff  ${HDF5_TOOLS_LIB_TARGET} ${HDF5_LIB_TARGET})
set_target_properties (h5diff PROPERTIES FOLDER tools)
set_global_variable (HDF5_UTILS_TO_EXPORT "${HDF5_UTILS_TO_EXPORT};h5diff")

set (H5_DEP_EXECUTABLES h5diff)

if (BUILD_SHARED_LIBS)
  add_executable (h5diff-shared
      ${HDF5_TOOLS_H5DIFF_SOURCE_DIR}/h5diff_common.c
      ${HDF5_TOOLS_H5DIFF_SOURCE_DIR}/h5diff_main.c
  )
  TARGET_NAMING (h5diff-shared SHARED)
  TARGET_C_PROPERTIES (h5diff-shared SHARED " " " ")
  target_link_libraries (h5diff-shared  ${HDF5_TOOLS_LIBSH_TARGET} ${HDF5_LIBSH_TARGET})
  set_target_properties (h5diff-shared PROPERTIES FOLDER tools)
  set_global_variable (HDF5_UTILS_TO_EXPORT "${HDF5_UTILS_TO_EXPORT};h5diff-shared")

  set (H5_DEP_EXECUTABLES ${H5_DEP_EXECUTABLES} h5diff-shared)
endif ()

if (H5_HAVE_PARALLEL)
  add_executable (ph5diff
      ${HDF5_TOOLS_H5DIFF_SOURCE_DIR}/h5diff_common.c
      ${HDF5_TOOLS_H5DIFF_SOURCE_DIR}/ph5diff_main.c
  )
  TARGET_NAMING (ph5diff STATIC)
  TARGET_C_PROPERTIES (ph5diff STATIC " " " ")
  target_link_libraries (ph5diff  ${HDF5_TOOLS_LIB_TARGET} ${HDF5_LIB_TARGET})
  set_target_properties (ph5diff PROPERTIES FOLDER tools)
  set_global_variable (HDF5_UTILS_TO_EXPORT "${HDF5_UTILS_TO_EXPORT};ph5diff")
endif ()

if (BUILD_TESTING)
  # --------------------------------------------------------------------
  # Add the h5diff and test executables
  # --------------------------------------------------------------------
  if (HDF5_BUILD_GENERATORS)
    add_executable (h5diffgentest ${HDF5_TOOLS_H5DIFF_SOURCE_DIR}/h5diffgentest.c)
    TARGET_NAMING (h5diffgentest STATIC)
    TARGET_C_PROPERTIES (h5diffgentest STATIC " " " ")
    target_link_libraries (h5diffgentest ${HDF5_LIB_TARGET})
    set_target_properties (h5diffgentest PROPERTIES FOLDER generator/tools)

    #add_test (NAME h5diffgentest COMMAND $<TARGET_FILE:h5diffgentest>)
  endif ()

  #-----------------------------------------------------------------------------
  # If plugin library tests can be tested
  #-----------------------------------------------------------------------------
  if (BUILD_SHARED_LIBS)
    set (HDF5_TOOL_PLUGIN_LIB_CORENAME         "dynlibdiff")
    set (HDF5_TOOL_PLUGIN_LIB_NAME             "${HDF5_EXTERNAL_LIB_PREFIX}${HDF5_TOOL_PLUGIN_LIB_CORENAME}")
    set (HDF5_TOOL_PLUGIN_LIB_TARGET           ${HDF5_TOOL_PLUGIN_LIB_CORENAME})
    add_definitions (${HDF_EXTRA_C_FLAGS})
    INCLUDE_DIRECTORIES (${HDF5_SRC_DIR})

    add_library (${HDF5_TOOL_PLUGIN_LIB_TARGET} SHARED dynlib_diff.c)
    TARGET_C_PROPERTIES (${HDF5_TOOL_PLUGIN_LIB_TARGET} SHARED " " " ")
    target_link_libraries (${HDF5_TOOL_PLUGIN_LIB_TARGET} ${HDF5_TEST_LIB_TARGET})
    H5_SET_LIB_OPTIONS (${HDF5_TOOL_PLUGIN_LIB_TARGET} ${HDF5_TOOL_PLUGIN_LIB_NAME} SHARED ${HDF5_PACKAGE_SOVERSION})

    # make plugins dir
    file (MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/plugins")
    #-----------------------------------------------------------------------------
    # Copy plugin library to a plugins folder
    #-----------------------------------------------------------------------------
    add_custom_command (
        TARGET     ${HDF5_TOOL_PLUGIN_LIB_TARGET}
        POST_BUILD
        COMMAND    ${CMAKE_COMMAND}
        ARGS       -E copy_if_different
            "$<TARGET_FILE:${HDF5_TOOL_PLUGIN_LIB_TARGET}>"
            "${CMAKE_BINARY_DIR}/plugins/$<TARGET_FILE_NAME:${HDF5_TOOL_PLUGIN_LIB_TARGET}>"
    )
  endif ()

  include (CMakeTests.cmake)

endif ()

##############################################################################
##############################################################################
###           I N S T A L L A T I O N                                      ###
##############################################################################
##############################################################################

#-----------------------------------------------------------------------------
# Rules for Installation of tools using make Install target
#-----------------------------------------------------------------------------
if (HDF5_EXPORTED_TARGETS)
  if (BUILD_SHARED_LIBS)
    INSTALL_PROGRAM_PDB (h5diff-shared ${HDF5_INSTALL_BIN_DIR} toolsapplications)
  endif ()
  INSTALL_PROGRAM_PDB (h5diff ${HDF5_INSTALL_BIN_DIR} toolsapplications)

  install (
      TARGETS
          ${H5_DEP_EXECUTABLES}
      EXPORT
          ${HDF5_EXPORTED_TARGETS}
      RUNTIME DESTINATION ${HDF5_INSTALL_BIN_DIR} COMPONENT toolsapplications
  )

  if (H5_HAVE_PARALLEL)
    #INSTALL_PROGRAM_PDB (ph5diff ${HDF5_INSTALL_BIN_DIR} toolsapplications)

    install (
        TARGETS
            ph5diff
        EXPORT
            ${HDF5_EXPORTED_TARGETS}
        RUNTIME DESTINATION ${HDF5_INSTALL_BIN_DIR} COMPONENT toolsapplications
    )
  endif ()
endif ()
