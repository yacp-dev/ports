cmake_minimum_required(VERSION 3.15)

project(libgdither VERSION 0.6 LANGUAGES C)

include(GNUInstallDirs)

add_library(gdither SHARED
    gdither.c
)

target_include_directories(gdither
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/libgdither>
)

set_target_properties(gdither PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    VERSION 1.0
    SOVERSION 1
)

target_compile_options(gdither PRIVATE -ffast-math)

# The original pkg-config file links -lm.
if(UNIX)
    target_link_libraries(gdither PRIVATE m)
endif()

install(TARGETS gdither
    EXPORT libgditherTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(FILES
    gdither.h
    gdither_types.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libgdither
)

# Generate libgdither.pc by taking the existing one in the source tree and
# replacing a few fields with CMake install variables.
set(_pc_source "${CMAKE_CURRENT_SOURCE_DIR}/libgdither.pc")
if(EXISTS "${_pc_source}")
    file(READ "${_pc_source}" _pc_contents)

    # Replace the path variables.
    string(REGEX REPLACE "prefix=[^\r\n]*" "prefix=${CMAKE_INSTALL_PREFIX}" _pc_contents "${_pc_contents}")
    string(REGEX REPLACE "libdir=[^\r\n]*" "libdir=${CMAKE_INSTALL_FULL_LIBDIR}" _pc_contents "${_pc_contents}")
    string(REGEX REPLACE "includedir=[^\r\n]*" "includedir=${CMAKE_INSTALL_FULL_INCLUDEDIR}" _pc_contents "${_pc_contents}")

    # Replace Version field.
    string(REGEX REPLACE "Version:[ \t]*[^\r\n]*" "Version: ${PROJECT_VERSION}" _pc_contents "${_pc_contents}")

    file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/libgdither.pc" "${_pc_contents}")
else()
    message(FATAL_ERROR "Expected pkg-config file not found: ${_pc_source}")
endif()

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/libgdither.pc"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)

# The tests the original tests/Makefile runs with "make test". They are
# built next to the DLL, so they find it without touching PATH.
include(CTest)
if(BUILD_TESTING)
    find_path(FFTW3_INCLUDE_DIR fftw3.h)
    find_library(FFTW3_LIBRARY fftw3)
    if(NOT FFTW3_INCLUDE_DIR OR NOT FFTW3_LIBRARY)
        message(FATAL_ERROR "fftw3 is required for the tests")
    endif()

    foreach(_test runcheck interleavetest distortion noisetest)
        add_executable(${_test} tests/${_test}.c)
        target_link_libraries(${_test} PRIVATE gdither m)
        add_test(NAME ${_test} COMMAND ${_test})
    endforeach()

    foreach(_test distortion noisetest)
        target_include_directories(${_test} PRIVATE ${FFTW3_INCLUDE_DIR})
        target_link_libraries(${_test} PRIVATE ${FFTW3_LIBRARY})
    endforeach()
endif()
