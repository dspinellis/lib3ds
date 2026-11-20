SET(LIB3DS_SOURCES
    lib3ds.h
    lib3ds_impl.h
    lib3ds_atmosphere.c
    lib3ds_background.c
    lib3ds_camera.c
    lib3ds_chunk.c
    lib3ds_chunktable.c
    lib3ds_file.c
    lib3ds_io.c
    lib3ds_light.c
    lib3ds_material.c
    lib3ds_math.c
    lib3ds_matrix.c
    lib3ds_mesh.c
    lib3ds_node.c
    lib3ds_quat.c
    lib3ds_shadow.c
    lib3ds_track.c
    lib3ds_util.c
    lib3ds_vector.c
    lib3ds_viewport.c)

IF(BUILD_SHARED_LIBS)
    # The version resource belongs in the DLL only; in a static library it
    # would clash with the version resource of whatever links against it.
    ADD_LIBRARY(lib3ds SHARED ${LIB3DS_SOURCES} lib3ds.rc)
    IF(MSVC)
        TARGET_COMPILE_DEFINITIONS(lib3ds PRIVATE LIB3DS_EXPORTS)
    ENDIF(MSVC)
ELSE(BUILD_SHARED_LIBS)
    ADD_LIBRARY(lib3ds STATIC ${LIB3DS_SOURCES})
    # PUBLIC so consumers (including the installed package) skip dllimport.
    TARGET_COMPILE_DEFINITIONS(lib3ds PUBLIC LIB3DS_STATIC)
    # Embed debug info in the .lib (/Z7) so consumers don't need a separate PDB.
    SET_TARGET_PROPERTIES(lib3ds PROPERTIES
        MSVC_DEBUG_INFORMATION_FORMAT "$<$<CONFIG:Debug,RelWithDebInfo>:Embedded>")
ENDIF(BUILD_SHARED_LIBS)

IF(WIN32)
    # Static lib gets an "s" suffix so it can't be mistaken for the DLL's
    # import library of the same name.
    IF(BUILD_SHARED_LIBS)
        SET(LIB3DS_OUTPUT_NAME "lib3ds-2_0")
    ELSE(BUILD_SHARED_LIBS)
        SET(LIB3DS_OUTPUT_NAME "lib3ds-2_0s")
    ENDIF(BUILD_SHARED_LIBS)
    SET_TARGET_PROPERTIES(lib3ds PROPERTIES
        OUTPUT_NAME ${LIB3DS_OUTPUT_NAME}
        DEBUG_POSTFIX "d")
ENDIF(WIN32)

SET_TARGET_PROPERTIES(lib3ds
    PROPERTIES VERSION 2.0)

TARGET_INCLUDE_DIRECTORIES(lib3ds PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

INSTALL(TARGETS lib3ds
    EXPORT lib3dsTargets
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

INSTALL(FILES lib3ds.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

IF(MSVC AND BUILD_SHARED_LIBS)
    INSTALL(FILES $<TARGET_PDB_FILE:lib3ds>
        DESTINATION ${CMAKE_INSTALL_BINDIR}
        CONFIGURATIONS Debug RelWithDebInfo
        OPTIONAL)
ENDIF(MSVC AND BUILD_SHARED_LIBS)
