cmake_minimum_required(VERSION 3.5...3.30)
project(x52pro
  VERSION 0.2.0
  DESCRIPTION "x52 PRO linux library"
  HOMEPAGE_URL "https://github.com/seesturm/x52lib"
  LANGUAGES C)

include(GNUInstallDirs)

find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBUSB REQUIRED libusb-1.0)

add_library(x52pro SHARED x52pro.c)
target_include_directories(x52pro
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  PRIVATE
    ${LIBUSB_INCLUDE_DIRS}
)
target_link_libraries(x52pro PRIVATE ${LIBUSB_LIBRARIES})
set_target_properties(x52pro PROPERTIES
  OUTPUT_NAME x52pro
  SOVERSION 0
  VERSION 0.2.0
  WINDOWS_EXPORT_ALL_SYMBOLS ON
)

install(TARGETS x52pro
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(FILES x52pro.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

add_executable(x52output x52output.c)
target_link_libraries(x52output PRIVATE x52pro)

install(TARGETS x52output RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES x52output.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)
install(FILES at.hasenleithner.plasma.x52pro.metainfo.xml DESTINATION ${CMAKE_INSTALL_DATADIR}/metainfo)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/x52pro.pc"
"prefix=${CMAKE_INSTALL_PREFIX}
exec_prefix=\${prefix}
libdir=\${exec_prefix}/${CMAKE_INSTALL_LIBDIR}
includedir=\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}

Name: x52pro
Description: x52 PRO linux library
Version: ${PROJECT_VERSION}
Requires.private: libusb-1.0
Libs: -L\${libdir} -lx52pro
Cflags: -I\${includedir}
")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/x52pro.pc" DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
