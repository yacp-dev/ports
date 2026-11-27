cmake_minimum_required(VERSION 3.16)

project(voro++ VERSION 0.4.6 LANGUAGES CXX)

set(SOVERSION 0)

include(GNUInstallDirs)

set(VORO_SOURCES
  src/cell.cc
  src/common.cc
  src/container.cc
  src/unitcell.cc
  src/v_compute.cc
  src/c_loops.cc
  src/v_base.cc
  src/wall.cc
  src/pre_container.cc
  src/container_prd.cc
)

add_library(voro++ ${VORO_SOURCES})
set_target_properties(voro++ PROPERTIES
  SOVERSION ${SOVERSION}
)
install(TARGETS voro++
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

add_executable(cmd_line src/cmd_line.cc)
target_link_libraries(cmd_line voro++)
set_target_properties(cmd_line PROPERTIES
  OUTPUT_NAME voro++
)
install(TARGETS cmd_line RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

include_directories(${CMAKE_SOURCE_DIR}/src)

set(VORO_HEADERS
  src/voro++.hh
  src/c_loops.hh
  src/cell.hh
  src/common.hh
  src/config.hh
  src/container.hh
  src/container_prd.hh
  src/rad_option.hh
  src/pre_container.hh
  src/unitcell.hh
  src/v_base.hh
  src/v_compute.hh
  src/wall.hh
  src/worklist.hh
)
install(FILES ${VORO_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/voro++)
install(FILES man/voro++.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/voro++.pc"
"prefix=${CMAKE_INSTALL_PREFIX}
exec_prefix=\${prefix}
libdir=\${exec_prefix}/${CMAKE_INSTALL_LIBDIR}
includedir=\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}

Name: voro++
Description: Library for 3D computations of the Voronoi tessellation
Version: ${PROJECT_VERSION}
Libs: -L\${libdir} -lvoro++
Cflags: -I\${includedir}
")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/voro++.pc"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
