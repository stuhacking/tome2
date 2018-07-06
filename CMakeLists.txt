## Tales of Middle Earth 2.3.9 continued...
##
cmake_minimum_required(VERSION 3.10)
project(tome2
    LANGUAGES C
    VERSION 2.3.9.0)

# Set up compiler options
set(CMAKE_C_STANDARD 11)
add_compile_options(-pipe -W -Wall -Wextra -Wno-unused-value -Wno-unused-parameter)
set(CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG")
set(CMAKE_C_FLAGS_DEBUG "-O0 -g")

#
# Set the path for loading the library bits.
#
if( SYSTEM_INSTALL )
  set(DEFAULT_PATH "${CMAKE_INSTALL_PREFIX}/lib/tome")
else()
  set(DEFAULT_PATH "./lib")
endif()

add_definitions(-DDEFAULT_PATH="${DEFAULT_PATH}")

# Add definitions.
add_definitions(-DUSE_EGO_GRAPHICS)
add_definitions(-DUSE_TRANSPARENCY)
add_definitions(-DSUPPORT_GAMMA)
add_definitions(-DUSE_PRECISE_CMOVIE)

# Which socket implementation are we using?
if( WIN32 )
  add_definitions(-DUSE_WINSOCK)
else()
  add_definitions(-DUSE_UNIXSOCK)
endif()

# Add the source subdirectory.
add_subdirectory(src/lua)
add_subdirectory(src)
add_subdirectory(lib)
