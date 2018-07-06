## tome2 libs and executables.
##

# Print a summary of enabled features and found packages.
include(FeatureSummary)

# Macro for defining tolua targets.
macro(tolua_file MODULE_NAME OUTPUT_FILE_NAME)
  add_custom_command(
      OUTPUT ${OUTPUT_FILE_NAME}
      COMMAND tolua -n ${MODULE_NAME} -o ${CMAKE_CURRENT_BINARY_DIR}/${OUTPUT_FILE_NAME} ${CMAKE_CURRENT_SOURCE_DIR}/${MODULE_NAME}.pkg
      DEPENDS tolua ${CMAKE_CURRENT_SOURCE_DIR}/${MODULE_NAME}.pkg
  )
  set_source_files_properties("${OUTPUT_FILE_NAME}" PROPERTIES GENERATED TRUE)
endmacro(tolua_file)

# Process all the needed modules.
tolua_file(monster  w_mnster.c)
tolua_file(player   w_player.c)
tolua_file(player_c w_play_c.c)
tolua_file(z_pack   w_z_pack.c)
tolua_file(object   w_obj.c)
tolua_file(util     w_util.c)
tolua_file(spells   w_spells.c)
tolua_file(quest    w_quest.c)
tolua_file(dungeon  w_dun.c)

set(SRC_COMMON
    z-rand.c z-util.c z-form.c z-virt.c z-term.c z-sock.c
    variable.c tables.c plots.c util.c cave.c dungeon.c
    melee1.c melee2.c modules.c
    object1.c object2.c randart.c squeltch.c traps.c
    monster1.c monster2.c monster3.c
    xtra1.c xtra2.c skills.c powers.c gods.c
    spells1.c spells2.c
    status.c files.c notes.c loadsave.c
    cmd1.c cmd2.c cmd3.c cmd4.c cmd5.c cmd6.c cmd7.c
    help.c
    generate.c gen_maze.c gen_evol.c wild.c levels.c store.c bldg.c
    cmovie.c irc.c
    wizard2.c init2.c birth.c wizard1.c init1.c
    # Lua bits:
    lua_bind.c script.c
    w_mnster.c w_player.c w_play_c.c w_z_pack.c
    w_obj.c w_util.c w_spells.c w_quest.c w_dun.c
    )

set(SRC_PLATFORM
    main-gcu.c main-x11.c main-xaw.c main-sdl.c main-gtk2.c
    )

# Need a few additional source files for Windows.
if( WIN32 )
  set(SRCS ${SRCS} main-win.c readdib.c)
  # Resource files require a little workaround.
  if( MINGW )
    # Workaround for resource compilation for mingw on CMake.
    # See http://www.cmake.org/Bug/view.php?id=4068
    add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/angband_rc.o
        COMMAND windres.exe -I${CMAKE_CURRENT_SOURCE_DIR}
        -i${CMAKE_CURRENT_SOURCE_DIR}/angband.rc
        -o ${CMAKE_CURRENT_BINARY_DIR}/angband_rc.o)
    set(SRCS ${SRCS} ${CMAKE_CURRENT_BINARY_DIR}/angband_rc.o)
  else( MINGW )
    set(SRCS ${SRCS} angband.rc)
  endif( MINGW )
endif( WIN32 )


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
    # SDL Port uses math.h
    set(LIBS ${LIBS} ${SDLIMAGE_LIBRARY} ${SDLTTF_LIBRARY} ${SDL_LIBRARY} m)
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

# Print out a summary of features.
feature_summary(WHAT ENABLED_FEATURES PACKAGES_FOUND DESCRIPTION "Enabled Features:")

## Tome static libraries
add_library(tomelib ${SRC_COMMON})
target_include_directories(tomelib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(tomelib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/lua)
target_link_libraries(tomelib PUBLIC lua) # Make math and lua available

add_library(tomelib-main main.c)
target_link_libraries(tomelib-main tomelib)

## Tome executable
add_executable(tome ${EXECUTABLE_OPTIONS} ${SRC_PLATFORM})
target_link_libraries(tome PRIVATE tomelib-main)
target_link_libraries(tome PRIVATE ${LIBS}) # front-end libs (See above for ports.)

## Installation
install(TARGETS tome
    RUNTIME DESTINATION bin
    )
