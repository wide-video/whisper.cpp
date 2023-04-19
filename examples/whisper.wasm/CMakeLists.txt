#
# libmain
#

set(TARGET libmain)

add_executable(${TARGET}
    emscripten.cpp
    )

include(DefaultTargetOptions)

target_link_libraries(${TARGET} PRIVATE
    common whisper
    )

unset(EXTRA_FLAGS)

if (WHISPER_WASM_SINGLE_FILE)
    set(EXTRA_FLAGS "-s SINGLE_FILE=1")
    message(STATUS "Embedding WASM inside main.js")

    add_custom_command(
        TARGET ${TARGET} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
        ${CMAKE_BINARY_DIR}/bin/libmain.js
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/whisper.wasm/main.js
        )
endif()

set_target_properties(${TARGET} PROPERTIES LINK_FLAGS " \
    --bind \
    -lworkerfs.js \
    -s INVOKE_RUN=0 \
    -s EXIT_RUNTIME=1 \
    -s MODULARIZE=1 \
    -s EXPORT_NAME=\"createWhisper\" \
    -s EXPORTED_FUNCTIONS=\"[_main, ___wasm_init_memory_flag]\" \
    -s EXPORTED_RUNTIME_METHODS=\"[out, err, ccall, cwrap, callMain, FS, WORKERFS]\" \
    -s INITIAL_MEMORY=128mb \
    -s ALLOW_MEMORY_GROWTH=1 \
    -s MAXIMUM_MEMORY=4gb \
    -s ENVIRONMENT=worker \
    -s PROXY_TO_PTHREAD=1 \
    -s STACK_SIZE=5MB \
    -s DEFAULT_PTHREAD_STACK_SIZE=2MB \
    -pthread \
    -s FORCE_FILESYSTEM=1 \
    ${EXTRA_FLAGS} \
    ")

message(EXTRA_FLAGS="${EXTRA_FLAGS}")

#
# whisper.wasm
#

set(TARGET whisper.wasm)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/index-tmpl.html  ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${TARGET}/index.html @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../helpers.js    ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${TARGET}/helpers.js @ONLY)
