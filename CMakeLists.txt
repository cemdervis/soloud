file(GLOB HeaderFiles "include/*.h")

file(GLOB SourceFiles
        "src/audiosource/**/*.h"
        "src/audiosource/**/*.cpp"
        "src/core/*.cpp"
        "src/filter/*.cpp"
)

add_library(SoLoud STATIC ${HeaderFiles} ${SourceFiles})

target_include_directories(SoLoud PUBLIC include)

set(SoLoud_BACKEND_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src/backend/")

if (EMSCRIPTEN)
    cerlib_Log("Linking SoLoud with SDL_mixer")

    target_compile_definitions(SoLoud PRIVATE -DWITH_SDL2_STATIC)
    target_sources(SoLoud PRIVATE ${SoLoud_BACKEND_DIR}/sdl2_static/soloud_sdl2_static.cpp)

    set(LinkFlags "SHELL:-s USE_SDL=2")
    target_compile_options(SoLoud PRIVATE ${LinkFlags})
    target_link_options(SoLoud PRIVATE ${LinkFlags})
    target_link_libraries(SoLoud PRIVATE SDL2)
elseif (WIN32)
    cerlib_Log("Linking SoLoud with XAudio2")

    target_compile_definitions(SoLoud PRIVATE -DWITH_XAUDIO2)
    target_sources(SoLoud PRIVATE ${SoLoud_BACKEND_DIR}/xaudio2/soloud_xaudio2.cpp)
elseif (APPLE)
    cerlib_Log("Linking SoLoud with CoreAudio")

    find_library(COREAUDIO_FRAMEWORK CoreAudio REQUIRED)
    find_library(AUDIOTOOLBOX_FRAMEWORK AudioToolbox REQUIRED)

    cerlib_Log("CoreAudio Framework: ${COREAUDIO_FRAMEWORK}")
    cerlib_Log("AudioToolbox Framework: ${AUDIOTOOLBOX_FRAMEWORK}")

    target_compile_definitions(SoLoud PRIVATE -DWITH_COREAUDIO)
    target_sources(SoLoud PRIVATE ${SoLoud_BACKEND_DIR}/coreaudio/soloud_coreaudio.cpp)

    target_link_libraries(SoLoud PUBLIC
            ${COREAUDIO_FRAMEWORK}
            ${AUDIOTOOLBOX_FRAMEWORK}
    )
elseif (ANDROID)
    cerlib_Log("Linking SoLoud with OpenSL ES")
    target_compile_definitions(SoLoud PRIVATE -DWITH_OPENSLES)
elseif (UNIX)
    cerlib_Log("Linking SoLoud with JACK")
    target_compile_definitions(SoLoud PRIVATE -DWITH_JACK)
    target_sources(SoLoud PRIVATE ${SoLoud_BACKEND_DIR}/jack/soloud_jack.cpp)
    target_link_libraries(SoLoud PUBLIC jack)
else ()
    message(FATAL_ERROR "Could not detect a proper backend for SoLoud!")
endif ()
