cmake_minimum_required(VERSION 3.5...3.30)
project(libxcorr
  VERSION 0
  DESCRIPTION "Cross Correlation function for C using FFTW"
  HOMEPAGE_URL "https://github.com/dMaggot/libxcorr"
  LANGUAGES C)

include(GNUInstallDirs)

find_package(PkgConfig REQUIRED)
pkg_check_modules(FFTW3 REQUIRED fftw3)

add_library(xcorr SHARED src/xcorr.c)
target_include_directories(xcorr
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  PRIVATE
    ${FFTW3_INCLUDE_DIRS}
)
target_link_libraries(xcorr PRIVATE ${FFTW3_LIBRARIES})
set_target_properties(xcorr PROPERTIES
  OUTPUT_NAME xcorr
  SOVERSION 0
)

install(TARGETS xcorr
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(FILES include/xcorr.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/libxcorr.pc"
"prefix=${CMAKE_INSTALL_PREFIX}
exec_prefix=\${prefix}
libdir=\${prefix}/${CMAKE_INSTALL_LIBDIR}
includedir=\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}

Name: libxcorr
Description: Cross Correlation function for C using FFTW
Version: ${PROJECT_VERSION}
Libs: -L\${libdir} -lxcorr
Cflags: -I\${includedir}
Requires: fftw3
")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libxcorr.pc" DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
