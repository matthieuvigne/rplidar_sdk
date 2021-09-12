cmake_minimum_required(VERSION 3.0)

#Configure project
project(rplidar VERSION 2.0)
set(PROJECT_DESCRIPTION "SDK for slamtech RPLidar")

# Option to choose between cross-compiling or compiling for local computer.
# If compiling for the current computer, _intel is appended to the library name.

option(CROSS_COMPILE "True to cross compile to arm, false to compile on current platform" ON)

# Set compiler and library name.

if(CROSS_COMPILE)
	set(CMAKE_CXX_COMPILER "arm-linux-gnueabihf-g++")
	set(LIBRARY_NAME ${PROJECT_NAME}_arm)
else()
	set(CMAKE_CXX_COMPILER "g++")
	set(LIBRARY_NAME ${PROJECT_NAME})
endif()

# Set compiler flags.
set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -fdiagnostics-color -lpthread -lm")


# Set build-type specific flags
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
	message("Build mode: debug.")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ggdb -DDEBUG")
else (CMAKE_BUILD_TYPE STREQUAL "Debug")
	message("Build mode: release build - optimization enabled.")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")
endif (CMAKE_BUILD_TYPE STREQUAL "Debug")

# Specify library content
file(GLOB ${PROJECT_NAME}_SOURCES "sdk/sdk/src/*.cpp")
file(GLOB ${PROJECT_NAME}_ARCH_SOURCES "sdk/sdk/src/arch/linux/*.cpp")
file(GLOB ${PROJECT_NAME}_HAL_SOURCES "sdk/sdk/src/hal/*.cpp")

# Add directory containing the headers, and put all headers in a variable.
include_directories(sdk/sdk/include)
include_directories(sdk/sdk/src)
file(GLOB_RECURSE ${PROJECT_NAME}_HEADERS "sdk/sdk/include/*.h")

# Setup static library
add_library(${LIBRARY_NAME} STATIC ${${PROJECT_NAME}_SOURCES} ${${PROJECT_NAME}_ARCH_SOURCES} ${${PROJECT_NAME}_HAL_SOURCES})
target_link_libraries(${LIBRARY_NAME})

# Create package config file from template.
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/rplidarTemplate.pc" "${CMAKE_CURRENT_BINARY_DIR}/${LIBRARY_NAME}.pc")

# Set install rules: copy library and headers.
install(TARGETS ${LIBRARY_NAME} DESTINATION "lib")
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/sdk/sdk/include/" DESTINATION "include")
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/sdk/sdk/src/hal/" DESTINATION "include/hal")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${LIBRARY_NAME}.pc" DESTINATION "lib/pkgconfig/")
