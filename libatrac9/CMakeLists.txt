cmake_minimum_required(VERSION 3.28)
project(libatrac9
  VERSION 0.0.3
  DESCRIPTION "Library for decoding audio in Sony's ATRAC9 format"
  HOMEPAGE_URL "https://github.com/Thealexbarney/LibAtrac9")

include(GNUInstallDirs)
include(CTest)

file(GLOB_RECURSE LIB_SOURCES "C/*.c")
add_library(atrac9 SHARED "${LIB_SOURCES}")
target_compile_definitions(atrac9 PUBLIC -DCOMPILING_DLL)
set_target_properties(atrac9 PROPERTIES SOVERSION 0)

install(TARGETS atrac9
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES C/libatrac9.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir ${CMAKE_INSTALL_PREFIX}/lib)
set(includedir ${CMAKE_INSTALL_PREFIX}/include)
configure_file(${CMAKE_SOURCE_DIR}/libatrac9.pc.in ${CMAKE_BINARY_DIR}/libatrac9.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/libatrac9.pc DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig)
