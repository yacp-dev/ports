cmake_minimum_required(VERSION 3.14)
project(linenoise
    VERSION 2.0
    DESCRIPTION "Guerrilla line editing library"
    HOMEPAGE_URL "https://github.com/antirez/linenoise"
    LANGUAGES C
)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# -----------------------------------------------------------------------
# Library target
# -----------------------------------------------------------------------
add_library(linenoise SHARED linenoise.c)
add_library(linenoise::linenoise ALIAS linenoise)

target_include_directories(linenoise
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

set_target_properties(linenoise PROPERTIES
    PUBLIC_HEADER linenoise.h
)

if (DEFINED SOVERSION)
    set_target_properties(linenoise PROPERTIES
        SOVERSION ${SOVERSION}
    )
endif()

# -----------------------------------------------------------------------
# Example executables (optional, built but not installed)
# -----------------------------------------------------------------------
add_executable(linenoise-example example.c)
target_link_libraries(linenoise-example PRIVATE linenoise)

add_executable(linenoise-test linenoise-test.c)
target_link_libraries(linenoise-test PRIVATE linenoise)

# -----------------------------------------------------------------------
# Installation
# -----------------------------------------------------------------------
install(TARGETS linenoise
    EXPORT  linenoiseTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# CMake package config
install(EXPORT linenoiseTargets
    FILE        linenoiseTargets.cmake
    NAMESPACE   linenoise::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/linenoise
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/linenoiseConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/linenoiseConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/linenoise
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/linenoiseConfigVersion.cmake"
    VERSION       ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/linenoiseConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/linenoiseConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/linenoise
)

# -----------------------------------------------------------------------
# pkg-config
# -----------------------------------------------------------------------
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/linenoise.pc.in"
    "${CMAKE_CURRENT_BINARY_DIR}/linenoise.pc"
    @ONLY
)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/linenoise.pc"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)
