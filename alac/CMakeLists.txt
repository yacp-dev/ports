cmake_minimum_required(VERSION 3.16)

project(alacconvert VERSION 0 LANGUAGES CXX)

include(GNUInstallDirs)

# Match the original Makefiles which compile even .c sources with g++.
set(ALAC_CODEC_C_SOURCES
    codec/EndianPortable.c
    codec/ALACBitUtilities.c
    codec/ag_dec.c
    codec/ag_enc.c
    codec/dp_dec.c
    codec/dp_enc.c
    codec/matrix_dec.c
    codec/matrix_enc.c
)

set(ALAC_CODEC_CPP_SOURCES
    codec/ALACDecoder.cpp
    codec/ALACEncoder.cpp
)

add_library(alac SHARED
    ${ALAC_CODEC_C_SOURCES}
    ${ALAC_CODEC_CPP_SOURCES}
)

set_source_files_properties(${ALAC_CODEC_C_SOURCES} PROPERTIES LANGUAGE CXX)

target_include_directories(alac
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/codec
)

# On Windows-like toolchains, exporting all symbols avoids needing explicit export macros.
set_target_properties(alac PROPERTIES
    VERSION 0
    SOVERSION 0
    WINDOWS_EXPORT_ALL_SYMBOLS ON
)

add_executable(alacconvert
    convert-utility/main.cpp
    convert-utility/CAFFileALAC.cpp
)

target_include_directories(alacconvert
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/codec
        ${CMAKE_CURRENT_SOURCE_DIR}/convert-utility
)

target_link_libraries(alacconvert PRIVATE alac)

install(TARGETS alac alacconvert
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(DIRECTORY codec/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/alac
    FILES_MATCHING PATTERN "*.h"
)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/alac.pc.in
    ${CMAKE_CURRENT_BINARY_DIR}/alac.pc
    @ONLY
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/alac.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)
