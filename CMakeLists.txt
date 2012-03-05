# @file /data/CMakeLists.txt
# @brief CMake file for the /data dirctory
# @author Joshua Gwinn (jdgbolt@gmail.com)
# @version
# @date 2012-02-03

# Include subdirectories in the tree for build process
ADD_SUBDIRECTORY(chapters)
ADD_SUBDIRECTORY(tilesets)
ADD_SUBDIRECTORY(images)
ADD_SUBDIRECTORY(music)
ADD_SUBDIRECTORY(sounds)

# Set variable to have a list of data files to copy over during build 
SET(DATA_FILES
   config.json
   ${CHAPTERS_FILES}
   ${TILESETS_FILES}
   ${IMAGES_FILES}
   ${MUSIC_FILES}
   ${SOUNDS_FILES}
)


# Copy each data file in turn into the data directory in the binary folder
FOREACH(DATA_FILE ${DATA_FILES})
   CONFIGURE_FILE(
      ${CMAKE_CURRENT_SOURCE_DIR}/${DATA_FILE}
      ${CMAKE_CURRENT_BINARY_DIR}/${DATA_FILE}
      COPYONLY)
ENDFOREACH()

# Set Cross Compilation settings to enable lz4comp to run when cross compiling
if (CMAKE_CROSSCOMPILING)
   SET(LZ4COMP_CROSSFILE "${CMAKE_BINARY_DIR}/../build/lz4comp_crossfile.cmake" CACHE FILEPATH
      "Location of lz4comp file to be used during cross compiling")
   INCLUDE(${LZ4COMP_CROSSFILE})
   SET(LZ4COMP_BINARY native-lz4comp)
else (CMAKE_CROSSCOMPILING)
   SET(LZ4COMP_BINARY lz4comp)
endif (CMAKE_CROSSCOMPILING)

# Set variable for textual data files to be compressed with LZ4
FILE(GLOB_RECURSE CONF_FILES RELATIVE
   ${CMAKE_CURRENT_BINARY_DIR} "${CMAKE_CURRENT_BINARY_DIR}/*.json")

# Compress each of the textual data files with lz4comp
FOREACH(CONF_FILE ${CONF_FILES})
   # Replace file extension with .lz4
   STRING(REGEX REPLACE ".json\$" ".lz4" CONF_FILE_OUT "${CONF_FILE}")
   # Add custom command to compress with lz4comp
   ADD_CUSTOM_COMMAND(
      OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/${CONF_FILE_OUT}"    # Output File
      COMMAND "${LZ4COMP_BINARY}" -jc                          # Command Name
      "${CMAKE_CURRENT_BINARY_DIR}/${CONF_FILE}"               # IN
      "${CMAKE_CURRENT_BINARY_DIR}/${CONF_FILE_OUT}"           # OUT
      DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/${CONF_FILE}"       # Depends on lz4comp
      "${LZ4COMP_BINARY}" COMMENT "Compress File: ${CONF_FILE}"
   )
   # Create a list of files to be used for the custom target
   SET(CONF_FILES_OUT ${CONF_FILES_OUT} "${CONF_FILE_OUT}")
ENDFOREACH(CONF_FILE)

# Add a custom target to have it compress at build time
ADD_CUSTOM_TARGET(CompressFiles ALL DEPENDS ${CONF_FILES_OUT})

# Install the files from the binary directory into the prefix
INSTALL(DIRECTORY 
   ${CMAKE_CURRENT_BINARY_DIR} 
   DESTINATION ${CMAKE_INSTALL_PREFIX}
   PATTERN "*.json" EXCLUDE
   PATTERN "Makefile" EXCLUDE
   PATTERN "CMakeFiles" EXCLUDE
   PATTERN "cmake_install.cmake" EXCLUDE
)

