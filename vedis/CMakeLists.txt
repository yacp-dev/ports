cmake_minimum_required(VERSION 3.28)
project(vedis
  VERSION 1.2.6
  DESCRIPTION "Embeddable Datastore Engine"
  HOMEPAGE_URL "https://vedis.symisc.net/")

include(GNUInstallDirs)

add_library(vedis SHARED vedis.c)
set_target_properties(vedis PROPERTIES SOVERSION 0)

install(TARGETS vedis
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES vedis.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix \${prefix})
set(libdir \${exec_prefix}/lib)
set(includedir \${prefix}/include)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/vedis.pc"
"prefix=${prefix}
exec_prefix=${exec_prefix}
libdir=${libdir}
includedir=${includedir}

Name: vedis
Description: ${PROJECT_DESCRIPTION}
Version: ${PROJECT_VERSION}
Libs: -L\${libdir} -lvedis
Cflags: -I\${includedir}
")
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/vedis.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
