cmake_minimum_required(VERSION 3.19)

set(CURRENT_PROJECT "OpenSimplex2")

option(OPENSIMPLEX_OLD "Enable build old C implementation instead Rust" OFF)

if (OPENSIMPLEX_OLD)

    set(SOURCE_CPP
        "${CMAKE_CURRENT_SOURCE_DIR}/_old/c/OpenSimplex2F.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/_old/c/OpenSimplex2F.h"
    )

    add_library(${CURRENT_PROJECT} ${SOURCE_CPP})
    target_include_directories(${CURRENT_PROJECT} PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/_old/c/")
    target_compile_options(${CURRENT_PROJECT} PRIVATE -Wall -Wextra --pedantic)

else()
    include(ExternalProject)

    ExternalProject_Add(
        ${CURRENT_PROJECT}Rust
        DOWNLOAD_COMMAND ""
        CONFIGURE_COMMAND ""
        BUILD_COMMAND cargo build COMMAND cargo build --release --features std --target-dir "${CMAKE_CURRENT_BINARY_DIR}"
        BINARY_DIR "${CMAKE_CURRENT_SOURCE_DIR}"
        INSTALL_COMMAND ""
        LOG_BUILD ON)


    # ADD_CUSTOM_TARGET(
    #     ${CURRENT_PROJECT}Rust
    #     COMMAND cargo build --release --features std --target-dir "${CMAKE_CURRENT_BINARY_DIR}"
    #     COMMENT "cargo build --release --features std --target-dir ${CMAKE_CURRENT_BINARY_DIR}"
    #     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    # )

    if(WIN32)
        if (BUILD_SHARED_LIBS)
            add_library( ${CURRENT_PROJECT} SHARED IMPORTED )

            # You can define two import-locations: one for debug and one for release.
            set_target_properties( ${CURRENT_PROJECT}
                PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/release/lib${CURRENT_PROJECT}.dll
                INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}"
                IMPORTED_IMPLIB ${CMAKE_CURRENT_BINARY_DIR}/release/lib${CURRENT_PROJECT}.lib
            )
        else()
            add_library( ${CURRENT_PROJECT} STATIC IMPORTED )

            # You can define two import-locations: one for debug and one for release.
            set_target_properties( ${CURRENT_PROJECT}
                PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/release/libopensimplex.lib
                INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}"

            )
        endif()

    elseif(APPLE)

        if (BUILD_SHARED_LIBS)
            add_library( ${CURRENT_PROJECT} SHARED IMPORTED )

            # You can define two import-locations: one for debug and one for release.
            set_target_properties( ${CURRENT_PROJECT}
                PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/release/libopensimplex.dylib
                INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}"

            )
        else()
            add_library( ${CURRENT_PROJECT} STATIC IMPORTED )

            # You can define two import-locations: one for debug and one for release.
            set_target_properties( ${CURRENT_PROJECT}
                PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/release/libopensimplex.lib
                INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}"

            )
        endif()

    elseif(UNIX)
        if (BUILD_SHARED_LIBS)
            add_library( ${CURRENT_PROJECT} SHARED IMPORTED )

            # You can define two import-locations: one for debug and one for release.
            set_target_properties( ${CURRENT_PROJECT}
                PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/release/libopensimplex.so
                INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}"
                IMPORTED_SONAME "lib${CURRENT_PROJECT}.so"

            )
        else()
            add_library( ${CURRENT_PROJECT} STATIC IMPORTED )

            # You can define two import-locations: one for debug and one for release.
            set_target_properties( ${CURRENT_PROJECT}
                PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/release/libopensimplex.a
                INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}"
                IMPORTED_SONAME "lib${CURRENT_PROJECT}.a"

            )
        endif()

    endif()

    add_dependencies(${CURRENT_PROJECT} ${CURRENT_PROJECT}Rust)


endif()
