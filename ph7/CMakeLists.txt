cmake_minimum_required(VERSION 3.28)
project(ph7
  VERSION 2.1.4
  DESCRIPTION "Embeddable PHP5 Engine"
  HOMEPAGE_URL "https://ph7.symisc.net/")

include(GNUInstallDirs)

add_library(ph7 SHARED ph7.c)
set_target_properties(ph7 PROPERTIES SOVERSION ${PROJECT_VERSION_MAJOR})

install(TARGETS ph7
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES ph7.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix \${prefix})
set(libdir \${exec_prefix}/lib)
set(includedir \${prefix}/include)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/ph7.pc"
"prefix=${prefix}
exec_prefix=${exec_prefix}
libdir=${libdir}
includedir=${includedir}

Name: ph7
Description: ${PROJECT_DESCRIPTION}
Version: ${PROJECT_VERSION}
Libs: -L\${libdir} -lph7
Cflags: -I\${includedir}
")
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/ph7.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
