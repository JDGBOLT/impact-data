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

# Install the files from the binary directory into the prefix
INSTALL(DIRECTORY 
${CMAKE_CURRENT_BINARY_DIR} 
DESTINATION ${CMAKE_INSTALL_PREFIX}
PATTERN "Makefile" EXCLUDE
PATTERN "CMakeFiles" EXCLUDE
PATTERN "cmake_install.cmake" EXCLUDE
)
