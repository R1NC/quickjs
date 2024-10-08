cmake_minimum_required(VERSION 3.17)
project(quickjs C)

set(CMAKE_C_STANDARD 99)

option(COMPILE_QUICKJS_QJSC "Compile qjsc or not" OFF)
option(COMPILE_QUICKJS_QJS "Compile qjs or not" OFF)
option(COMPILE_QUICKJS_TEST "Compile test or not" OFF)

if(USE_LUA)
        add_definitions(-DUSE_LUA)
endif()

file (STRINGS ${CMAKE_CURRENT_SOURCE_DIR}/VERSION BUILD_VERSION)

set(SOURCES_ROOT ${CMAKE_CURRENT_SOURCE_DIR})
set(SOURCES
        ${SOURCES_ROOT}/quickjs.h
        ${SOURCES_ROOT}/quickjs-libc.h
        ${SOURCES_ROOT}/quickjs.c
        ${SOURCES_ROOT}/libregexp.c
        ${SOURCES_ROOT}/libunicode.c
        ${SOURCES_ROOT}/libbf.c
        ${SOURCES_ROOT}/cutils.c
        ${SOURCES_ROOT}/quickjs-libc.c
)

IF (WIN32)
        list(APPEND SOURCES ${SOURCES_ROOT}/win_patch.c)
ENDIF()

add_library(quickjs STATIC ${SOURCES})
target_compile_definitions(${PROJECT_NAME}
        PRIVATE
        CONFIG_VERSION="${BUILD_VERSION}"
)

if (COMPILE_QUICKJS_QJSC)
        set(QJSC_SOURCES ${SOURCES_ROOT}/qjsc.c)

        IF (WIN32)
                list(APPEND QJSC_SOURCES ${SOURCES_ROOT}/getopt.c)
                list(APPEND QJSC_SOURCES ${SOURCES_ROOT}/win_patch.c)
        ENDIF()

        add_executable(qjsc ${QJSC_SOURCES})

        target_link_libraries(qjsc PRIVATE quickjs)

        target_compile_definitions(qjsc
                PRIVATE
                CONFIG_VERSION="${BUILD_VERSION}"
        )

        add_custom_command(OUTPUT repl.c COMMAND qjsc -c -o repl.c -m ${SOURCES_ROOT}/repl.js)
endif()

if (COMPILE_QUICKJS_QJS)
        set(QJS_SOURCES ${SOURCES_ROOT}/qjs.c repl.c)

        IF (WIN32)
                list(APPEND QJS_SOURCES ${SOURCES_ROOT}/win_patch.c)
        ENDIF()

        add_executable(qjs ${QJS_SOURCES})

        target_link_libraries(qjs PRIVATE quickjs)

        target_compile_definitions(qjs
                PRIVATE
                CONFIG_VERSION="${BUILD_VERSION}"
        )
endif()

if (COMPILE_QUICKJS_TEST)
        set(TEST262_SOURCES ${SOURCES_ROOT}/run-test262.c)

        IF (WIN32)
                list(APPEND TEST262_SOURCES ${SOURCES_ROOT}/win_patch.c)
        ENDIF()

        add_executable(run-test262 ${TEST262_SOURCES})

        target_link_libraries(run-test262 PRIVATE quickjs)

        target_compile_definitions(run-test262
                PRIVATE
                CONFIG_VERSION="${BUILD_VERSION}"
        )
endif()