cmake_minimum_required(VERSION 3.28)
project(fpconv
  VERSION 0
  DESCRIPTION "Fast and accurate double to string conversion"
  HOMEPAGE_URL "https://github.com/night-shift/fpconv")

include(GNUInstallDirs)

add_library(fpconv SHARED src/fpconv.c)
set_target_properties(fpconv PROPERTIES SOVERSION 0)

install(TARGETS fpconv
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES src/fpconv.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir ${CMAKE_INSTALL_PREFIX}/lib)
set(includedir ${CMAKE_INSTALL_PREFIX}/include)
configure_file(${CMAKE_SOURCE_DIR}/fpconv.pc.in ${CMAKE_BINARY_DIR}/fpconv.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/fpconv.pc DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig)
