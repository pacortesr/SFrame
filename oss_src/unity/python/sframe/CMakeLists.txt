project(unity)


# The key things to set up here are 
#  - INSTALLATION_BINARY_FILES: 
#     files to copy across to the python build. This must reference absolute 
#     paths of files to copy.
#  - INSTALLATION_EXTENSIONS
#     extensions to copy across. This must reference extension targets 
#     added byh make_extension in the extensions/ subdirectory.
#  - INSTALLATION_BINARY_TARGETS
#     targets to copy across. This references any executable / library target
#     with a single output file.
#
#  These variables are set as directory properties so they persist.

if(NOT WIN32)
  set(INSTALLATION_BINARY_FILES
    ${CMAKE_SOURCE_DIR}/deps/local/lib/libhdfs.so
    ${CMAKE_CURRENT_BINARY_DIR}/../../../sframe/spark_unity.jar)
else()
  set(INSTALLATION_BINARY_FILES
    ${CMAKE_SOURCE_DIR}/deps/local/lib/libhdfs.so)
endif()

# extensions
set(INSTALLATION_EXTENSIONS)
get_property(EXTENSIONS_LIST DIRECTORY "../../extensions" PROPERTY EXTENSIONS_LIST)
foreach(loop_var ${EXTENSIONS_LIST})
  message(STATUS "Copying Extension to graphlab python directory ${loop_var}")
  list(APPEND INSTALLATION_EXTENSIONS ${loop_var})
endforeach()

set(INSTALLATION_BINARY_TARGETS
  unity_server
  ## spark_unity
  minipsutil
)

make_copy_target(release_binaries ALL
        TARGETS
        ${INSTALLATION_BINARY_TARGETS}
        ${UNITY_ALL_LIBRARIES}
        ${INSTALLATION_EXTENSIONS}
        FILES
        ${INSTALLATION_BINARY_FILES}
        ${INSTALLATION_SYSTEM_BINARY_FILES}
        )

ADD_CUSTOM_TARGET (oss_clean_graphlab_binaries)
ADD_CUSTOM_COMMAND(
  COMMENT "clean graphlab binaries"
  COMMAND rm -f ${INSTALLATION_BINARY_TARGETS} ${UNITY_ALL_LIBRARIES} *.so *.dylib *.dll *.jar
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
  TARGET oss_clean_graphlab_binaries 
)

add_subdirectory(cython)
add_subdirectory(_spark)


set_property(DIRECTORY PROPERTY INSTALLATION_EXTENSIONS "${INSTALLATION_EXTENSIONS}")
set_property(DIRECTORY PROPERTY INSTALLATION_BINARY_TARGETS "${INSTALLATION_BINARY_TARGETS}")
set_property(DIRECTORY PROPERTY INSTALLATION_BINARY_FILES "${INSTALLATION_BINARY_FILES}")

