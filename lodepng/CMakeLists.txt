cmake_minimum_required(VERSION 3.28)
project(lodepng
  VERSION 0
  DESCRIPTION "PNG image decoder and encoder"
  HOMEPAGE_URL "https://github.com/lvandeve/lodepng")

include(GNUInstallDirs)
include(CTest)

add_library(lodepng SHARED "lodepng.cpp")
set_target_properties(lodepng PROPERTIES SOVERSION 0)
install(TARGETS lodepng
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES lodepng.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

add_executable(pngdetail "lodepng_util.cpp" "pngdetail.cpp")
target_link_libraries(pngdetail PRIVATE lodepng)
install(TARGETS pngdetail RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

find_package(SDL2 REQUIRED)

add_executable(benchmark "lodepng_util.cpp" "lodepng_benchmark.cpp")
target_link_libraries(benchmark PRIVATE lodepng SDL2::SDL2)

add_executable(unittest "lodepng_util.cpp" "lodepng_unittest.cpp")
target_link_libraries(unittest PRIVATE lodepng)
add_test(NAME unittest COMMAND unittest)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/lodepng.pc"
"prefix=${CMAKE_INSTALL_PREFIX}
exec_prefix=\${prefix}
libdir=\${exec_prefix}/${CMAKE_INSTALL_LIBDIR}
includedir=\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}

Name: lodepng
Description: PNG image decoder and encoder
Version: ${PROJECT_VERSION}
Libs: -L\${libdir} -llodepng
Cflags: -I\${includedir}
")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/lodepng.pc"
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
