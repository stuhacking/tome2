## Tales of Middle Earth 2.3.9 continued...
##
cmake_minimum_required(VERSION 3.10)
project(tome2
    LANGUAGES C
    VERSION 2.3.9.0)

# We want a readable feature summary.
include(FeatureSummary)

set(CMAKE_C_STANDARD 11)
add_compile_options(-pipe -W -Wall -Wextra -Wno-unused-value -Wno-unused-parameter)

set(CMAKE_C_FLAGS_RELEASE "-O2")
set(CMAKE_C_FLAGS_DEBUG "-O0 -g")


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


#
# X11 support (OPTIONAL)
#
find_package(X11)
if( X11_FOUND )
  # Add X11 flags/options
  add_definitions(-DUSE_X11)
  include_directories(${X11_INCLUDE_DIR})
  set(LIBS ${LIBS} ${X11_LIBRARIES})
endif()


#
# GTK2 support (OPTIONAL)
#
find_package(GTK2)
if( GTK2_FOUND )
  # Add GTK flags/options
  add_definitions(-DUSE_GTK2)
  include_directories(${GTK2_INCLUDE_DIRS})
  set(LIBS ${LIBS} ${GTK2_LIBRARIES})
endif()


#
# SDL support (OPTIONAL)
#
find_package(SDL)
if( SDL_FOUND )
  # This is a bit roundabout, but we're working around
  # the FindSDL_* scripts not respecting the REQUIRED
  # flag.
  #
  # the SDL port also requires SDL_image and SDL_ttf
  find_package(SDL_image)
  find_package(SDL_ttf)
  if( SDLIMAGE_FOUND AND SDLTTF_FOUND )
    # Add SDL flags/options
    add_definitions(-DUSE_SDL)
    include_directories(${SDL_INCLUDE_DIR} ${SDLIMAGE_INCLUDE_DIR} ${SDLTTF_INCLUDE_DIR})
    set(LIBS ${LIBS} ${SDLIMAGE_LIBRARY} ${SDLTTF_LIBRARY} ${SDL_LIBRARY})
  else()
    # Let user know that (and why) we haven't enabled SDL.
    if( SDLIMAGE_FOUND )
      message(STATUS "Found SDL and SDL_image, but not SDL_ttf!")
    elseif( SDLTTF_FOUND )
      message(STATUS "Found SDL and SDL_ttf, but not SDL_image!")
    else()
      message(STATUS "Found SDL, but not SDL_image nor SDL_ttf!")
    endif()
    # add info about finding but not enabling SDL
    set_feature_info(SDL "not enabled")
  endif()
endif()


#
# Curses support (OPTIONAL)
#
find_package(Curses)
if( CURSES_FOUND )
  # Add Curses flags/options
  add_definitions(-DUSE_GCU)
  include_directories(${CURSES_INCLUDE_DIR})
  set(LIBS ${LIBS} ${CURSES_LIBRARIES})
endif()


#
# Windows support
#
if( WIN32 )
  # Add Windows flags/options
  add_definitions(-DWINDOWS)
  set(EXECUTABLE_OPTIONS WIN32)
  set(LIBS ${LIBS} winmm wsock32)
endif( WIN32 )


#
# Set the path for loading the library bits.
#
if( SYSTEM_INSTALL )
  set(DEFAULT_PATH "${CMAKE_INSTALL_PREFIX}/lib/tome")
else()
  set(DEFAULT_PATH "./lib")
endif()
add_definitions(-DDEFAULT_PATH="${DEFAULT_PATH}")


# Print out a summary of features.
feature_summary(WHAT ENABLED_FEATURES PACKAGES_FOUND DESCRIPTION "Enabled Features:")

# Add the source subdirectory.
add_subdirectory(src)
add_subdirectory(lib)
