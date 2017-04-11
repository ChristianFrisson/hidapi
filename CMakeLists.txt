project(hidapi)
cmake_minimum_required(VERSION 3.6)
set(TARGET_NAME "hidapi")

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/hidapi)

set(${TARGET_NAME}_HDRS "${CMAKE_CURRENT_SOURCE_DIR}/hidapi/hidapi.h")

# sources
if(APPLE)
	list(APPEND ${TARGET_NAME}_SRCS "${CMAKE_CURRENT_SOURCE_DIR}/mac/hid.c")
	add_definitions(-DOS_DARWIN)
elseif(UNIX)
	list(APPEND ${TARGET_NAME}_SRCS "${CMAKE_CURRENT_SOURCE_DIR}/libusb/hid.c")
	add_definitions(-DOS_LINUX)
elseif(WIN32 OR MINGW)
	list(APPEND ${TARGET_NAME}_SRCS "${CMAKE_CURRENT_SOURCE_DIR}/windows/hid.c")
	add_definitions(-DOS_WINDOWS)
endif()

# create lib
add_library(${TARGET_NAME} ${${TARGET_NAME}_SRCS} ${${TARGET_NAME}_HDRS})

# link to dependencies
if(APPLE)
	target_link_libraries(${TARGET_NAME} "-framework IOKit -framework CoreFoundation")
elseif(UNIX)
	# NB (linux): requires libusb-1.0, not libusb-0.1 (i.e. don't install libusb-dev)
	target_link_libraries(${TARGET_NAME} ${LIBUSB_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
elseif(WIN32 OR MINGW)
	target_link_libraries(${TARGET_NAME} -lsetupapi)
endif()

set_target_properties("${TARGET_NAME}" PROPERTIES FOLDER "${FOLDER_NAME}")

# create test executable
add_executable(hidtest ${CMAKE_CURRENT_SOURCE_DIR}/hidtest/hidtest.cpp)
target_link_libraries(hidtest ${TARGET_NAME})
set_target_properties(hidtest PROPERTIES FOLDER "${FOLDER_NAME}")

if(CMAKE_CAN_APPEND_PROPERTIES)
	set_property(TARGET "${TARGET_NAME}" APPEND PROPERTY INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/hidapi;${CMAKE_CURRENT_SOURCE_DIR}/src/include")
endif()
