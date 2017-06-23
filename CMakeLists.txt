cmake_minimum_required(VERSION 3.1)

project(obs-studio-node VERSION 1.0.0)

set(
    OSN_NODE_NAME "obs_node"
    CACHE PATH "Name of the node binary output"
)

set(
    OSN_DIST_DIR "node-obs"
    CACHE PATH "Name of distribution directory"
)

set(
    OBS_BUILD_TYPE "Release"
    CACHE STRING "Configuration of OBS binaries to copy"
)

set(
    OBS_STUDIO_PATH "${CMAKE_SOURCE_DIR}/.." 
    CACHE PATH "Path to 64-bit build directory"
)

set(
    OBS_STUDIO_BUILD64 "${CMAKE_SOURCE_DIR}/../build64" 
    CACHE PATH "Path to 64-bit build directory"
)

set(
    OBS_STUDIO_DEPS64 "${CMAKE_SOURCE_DIR}/../../obs-studio-deps/win64"
    CACHE PATH "Path to 64-bit dependencies for obs studio"
)

if(IS_DIRECTORY ${OBS_STUDIO_PATH})
    message("OBS Studio 64-bit Build Path: ${OBS_STUDIO_BUILD64}")
else()
    message(FATAL_ERROR 
        "Failed to find OBS Studio using OBS_STUDIO_PATH: ${OBS_STUDIO_PATH}"
    )
endif()

add_library(${OSN_NODE_NAME} SHARED
    "src/SceneItem.cpp"
    "src/SceneItem.h"
    "src/Scene.cpp"
    "src/Scene.h"
    "src/IEncoder.cpp"
    "src/IEncoder.h"
    "src/Display.cpp"
    "src/Display.h"
    "src/RenderPlugin.h"
    "src/RenderPlugin.c"
    "src/Global.cpp"
    "src/Global.h"
    "src/Input.cpp"
    "src/Input.h"
    "src/ISource.cpp"
    "src/ISource.h"
    "src/Module.cpp"
    "src/Module.h"
    "src/Audio.cpp"
    "src/Audio.h"
    "src/Video.cpp"
    "src/Video.h"
    "src/obs-node.cpp"
    "src/obspp/obspp.hpp"
    "src/obspp/obspp.cpp"
    "src/obspp/obspp-audio.hpp"
    "src/obspp/obspp-audio.cpp"
    "src/obspp/obspp-display.hpp"
    "src/obspp/obspp-display.cpp"
    "src/obspp/obspp-encoder.hpp"
    "src/obspp/obspp-encoder.cpp"
    "src/obspp/obspp-filter.hpp"
    "src/obspp/obspp-filter.cpp"
    "src/obspp/obspp-index.hpp"
    "src/obspp/obspp-input.hpp"
    "src/obspp/obspp-input.cpp"
    "src/obspp/obspp-module.hpp"
    "src/obspp/obspp-module.cpp"
    "src/obspp/obspp-output.hpp"
    "src/obspp/obspp-scene.hpp"
    "src/obspp/obspp-scene.cpp"
    "src/obspp/obspp-service.hpp"
    "src/obspp/obspp-source.hpp"
    "src/obspp/obspp-source.cpp"
    "src/obspp/obspp-transition.hpp"
    "src/obspp/obspp-video.hpp"
    "src/obspp/obspp-video.cpp"
)

target_link_libraries(${OSN_NODE_NAME} ${CMAKE_JS_LIB} obs)

add_library(simple_draw SHARED
    "src/simple_draw/draw.c")

target_link_libraries(simple_draw
    ${OSN_NODE_NAME}
    obs)

target_compile_definitions(
    ${OSN_NODE_NAME} PUBLIC
    "BUILDING_NODE_EXTENSION"
)

target_include_directories(
    ${OSN_NODE_NAME} PUBLIC
    "${CMAKE_JS_INC}"
    "${OBS_STUDIO_PATH}/libobs"
)

set_target_properties(${OSN_NODE_NAME} PROPERTIES PREFIX "" SUFFIX ".node")

add_library(obs SHARED IMPORTED)

if (WIN32)
    set_target_properties(obs PROPERTIES
        IMPORTED_LOCATION_DEBUG "${OBS_STUDIO_BUILD64}/libobs/Debug/obs.dll"
        IMPORTED_IMPLIB_DEBUG "${OBS_STUDIO_BUILD64}/libobs/Debug/obs.lib"
        IMPORTED_LOCATION_RELEASE "${OBS_STUDOBS_STUDIO_BUILD64_PATH}/libobs/Release/obs.dll"
        IMPORTED_IMPLIB_RELEASE "${OBS_STUDIO_BUILD64}/libobs/Release/obs.lib"
    )
elseif()
    message(FATAL_ERROR "Only Windows supported for right now")
endif()

# Just copy all dependencies into a local folder so we can run our tests.
set( OBS_DEPENDENCIES 
    "${OBS_STUDIO_DEPS64}/bin/avcodec-57.dll"
    "${OBS_STUDIO_DEPS64}/bin/avdevice-57.dll"
    "${OBS_STUDIO_DEPS64}/bin/avfilter-6.dll"
    "${OBS_STUDIO_DEPS64}/bin/avformat-57.dll"
    "${OBS_STUDIO_DEPS64}/bin/avutil-55.dll"
    "${OBS_STUDIO_DEPS64}/bin/libcurl.dll"
    "${OBS_STUDIO_DEPS64}/bin/libogg-0.dll"
    "${OBS_STUDIO_DEPS64}/bin/libopus-0.dll"
    "${OBS_STUDIO_DEPS64}/bin/libpng16-16.dll"
    "${OBS_STUDIO_DEPS64}/bin/libvorbis-0.dll"
    "${OBS_STUDIO_DEPS64}/bin/libvorbisenc-2.dll"
    "${OBS_STUDIO_DEPS64}/bin/libvorbisfile-3.dll"
    "${OBS_STUDIO_DEPS64}/bin/libvpx-1.dll"
    "${OBS_STUDIO_DEPS64}/bin/libx264-148.dll"
    "${OBS_STUDIO_DEPS64}/bin/swresample-2.dll"
    "${OBS_STUDIO_DEPS64}/bin/swscale-4.dll"
    "${OBS_STUDIO_DEPS64}/bin/zlib.dll"
    "${OBS_STUDIO_BUILD64}/libobs/${OBS_BUILD_TYPE}/obs.dll"
    "${OBS_STUDIO_BUILD64}/libobs-d3d11/${OBS_BUILD_TYPE}/libobs-d3d11.dll"
    "${OBS_STUDIO_BUILD64}/deps/glad/${OBS_BUILD_TYPE}/obsglad.dll"
    "${OBS_STUDIO_BUILD64}/deps/w32-pthreads/${OBS_BUILD_TYPE}/w32-pthreads.dll"
)

add_custom_command(TARGET ${OSN_NODE_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}")

foreach( DEPENDENCY ${OBS_DEPENDENCIES})
    add_custom_command(TARGET ${OSN_NODE_NAME} POST_BUILD 
        COMMAND ${CMAKE_COMMAND}
        ARGS -E copy ${DEPENDENCY} "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}/"
    )
endforeach()

add_custom_command(TARGET ${OSN_NODE_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory 
    "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}/data")

add_custom_command(TARGET ${OSN_NODE_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory 
    "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}/data/libobs")

add_custom_command(TARGET ${OSN_NODE_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory 
    "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}/data/obs-plugins")

function(copy_obs_plugin PLUGIN_NAME)
    install(
        DIRECTORY "${OBS_STUDIO_PATH}/plugins/${PLUGIN_NAME}/data"
        DESTINATION "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}/data/obs-plugins/${PLUGIN_NAME}"
    )

    install(
        FILES
        "${OBS_STUDIO_BUILD64}/plugins/${PLUGIN_NAME}/Debug/${PLUGIN_NAME}.dll"
        "${OBS_STUDIO_BUILD64}/plugins/${PLUGIN_NAME}/Debug/${PLUGIN_NAME}.pdb"
        DESTINATION "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}/obs-plugins"
    )
endfunction()

file(GLOB DATA_LIBOBS
    "${OBS_STUDIO_PATH}/libobs/data/*")

foreach( DATA_FILES ${DATA_LIBOBS})
    add_custom_command(TARGET ${OSN_NODE_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND}
        ARGS -E copy ${DATA_FILES} "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}/data/libobs")
endforeach()

copy_obs_plugin("image-source")
copy_obs_plugin("obs-ffmpeg")
copy_obs_plugin("obs-filters")
copy_obs_plugin("obs-outputs")
copy_obs_plugin("obs-qsv11")
copy_obs_plugin("obs-text")
copy_obs_plugin("obs-transitions")
copy_obs_plugin("obs-x264")
copy_obs_plugin("rtmp-services")
copy_obs_plugin("text-freetype2")
copy_obs_plugin("win-capture")
copy_obs_plugin("win-dshow")
copy_obs_plugin("win-mf")
copy_obs_plugin("win-wasapi")

install(TARGETS ${OSN_NODE_NAME} simple_draw DESTINATION "${CMAKE_SOURCE_DIR}/${OSN_DIST_DIR}")