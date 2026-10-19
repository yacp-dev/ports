cmake_minimum_required(VERSION 3.28)
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/sml.pc" SML_PC_VERSION REGEX "^Version:")
string(REGEX REPLACE "^Version:[ \t]*" "" SML_PC_VERSION "${SML_PC_VERSION}")
project(libsml
  VERSION ${SML_PC_VERSION}
  DESCRIPTION "Implementation in C of the Smart Message Language (SML) protocol"
  HOMEPAGE_URL "https://github.com/volkszaehler/libsml")

include(GNUInstallDirs)
include(CTest)

find_package(pkgconfig REQUIRED)
pkg_check_modules(UUID uuid REQUIRED)

file(GLOB_RECURSE LIB_SOURCES "sml/src/*.c")
add_library(sml SHARED "${LIB_SOURCES}")
target_include_directories(sml PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/sml/include")
target_link_libraries(sml PUBLIC ${UUID_LIBRARIES})
set_target_properties(sml PROPERTIES SOVERSION 1)

file(GLOB_RECURSE BIN_SOURCES "examples/*.c")
add_executable(sml_server "${BIN_SOURCES}")
target_link_libraries(sml_server sml)
install(TARGETS sml_server RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

file(GLOB_RECURSE TEST_SOURCES "test/*.c")
add_executable(test_main "${TEST_SOURCES}")
target_link_libraries(test_main sml)
add_test(NAME test_main COMMAND test_main)

install(TARGETS sml
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(DIRECTORY sml/include/sml DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir ${CMAKE_INSTALL_PREFIX}/lib)
set(includedir ${CMAKE_INSTALL_PREFIX}/include)
configure_file(${CMAKE_SOURCE_DIR}/libsml.pc.in ${CMAKE_BINARY_DIR}/libsml.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/libsml.pc DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig)
