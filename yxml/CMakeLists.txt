cmake_minimum_required(VERSION 3.28)
project(yxml
  VERSION 0
  DESCRIPTION "Small, fast and mostly-correct XML parser"
  HOMEPAGE_URL "https://dev.yorhel.nl/yxml"
  LANGUAGES C)

include(GNUInstallDirs)
enable_testing()

add_library(yxml SHARED yxml.c)
target_include_directories(yxml PUBLIC ${CMAKE_SOURCE_DIR})
set_target_properties(yxml PROPERTIES SOVERSION 0)

install(TARGETS yxml
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES yxml.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir ${CMAKE_INSTALL_PREFIX}/lib)
set(includedir ${CMAKE_INSTALL_PREFIX}/include)
set(VERSION ${PROJECT_VERSION})
configure_file(${CMAKE_SOURCE_DIR}/yxml.pc.in ${CMAKE_BINARY_DIR}/yxml.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/yxml.pc DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig)

# test/test.sh runs ./test on each test/*.xml and compares with test/*.out
add_executable(yxml_test test/test.c)
target_link_libraries(yxml_test PRIVATE yxml)
set_target_properties(yxml_test PROPERTIES
  OUTPUT_NAME test
  RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/test)
file(GLOB yxml_test_data ${CMAKE_SOURCE_DIR}/test/*.xml ${CMAKE_SOURCE_DIR}/test/*.out ${CMAKE_SOURCE_DIR}/test/test.sh)
file(COPY ${yxml_test_data} DESTINATION ${CMAKE_BINARY_DIR}/test)
add_test(NAME yxml COMMAND sh test.sh WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/test)
set_tests_properties(yxml PROPERTIES
  ENVIRONMENT "PATH=$<TARGET_FILE_DIR:yxml>:$ENV{PATH}")
