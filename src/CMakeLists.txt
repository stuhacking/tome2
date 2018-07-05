## tome2 libs and executables.
##

# Lua support code.
add_subdirectory(lua)

set(SRCS
    main-gcu.c main-x11.c main-xaw.c main-sdl.c main-gtk2.c
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
    wizard2.c init2.c birth.c wizard1.c init1.c main.c
    # Lua bits:
    lua_bind.c script.c
    w_mnster.c w_player.c w_play_c.c w_z_pack.c
    w_obj.c w_util.c w_spells.c w_quest.c w_dun.c
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

# tome executable
add_executable(tome ${EXECUTABLE_OPTIONS} ${SRCS})
target_include_directories(tome PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(tome PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/lua)
target_link_libraries(tome PRIVATE m lua ${LIBS}) # math, lua, front-end libs (See /CMakeLists.txt)

# Installation
install(TARGETS tome
    RUNTIME DESTINATION bin
    )
