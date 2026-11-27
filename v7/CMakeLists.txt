cmake_minimum_required(VERSION 3.10)
project(v7 VERSION 3.0 LANGUAGES C)

include(GNUInstallDirs)

add_library(v7 SHARED v7.c)
set_target_properties(v7 PROPERTIES
  SOVERSION ${PROJECT_VERSION_MAJOR}
  OUTPUT_NAME v7
)
target_compile_definitions(v7 PRIVATE CS_ENABLE_UTF8 _GNU_SOURCE)
target_include_directories(v7 PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

install(TARGETS v7
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(FILES v7.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/v7.pc"
"prefix=${CMAKE_INSTALL_PREFIX}
exec_prefix=\${prefix}
libdir=\${exec_prefix}/${CMAKE_INSTALL_LIBDIR}
includedir=\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}

Name: v7
Description: Embedded JavaScript engine for C/C++
Version: ${PROJECT_VERSION}
Libs: -L\${libdir} -lv7
Cflags: -I\${includedir}
")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/v7.pc"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

include(CTest)
if(BUILD_TESTING)
  add_executable(unit_test tests/unit_test.c tests/common/test_util.c)
  target_compile_definitions(unit_test PRIVATE
    V7_EXPOSE_PRIVATE
    V7_UNIT_TEST
    V7_FILENAMES_SUPPRESS_CFUNC_ADDR
    _GNU_SOURCE
  )
  target_include_directories(unit_test PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
  add_test(NAME unit_test COMMAND unit_test)
  set_tests_properties(unit_test PROPERTIES
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/tests"
    ENVIRONMENT "TZ=UTC"
  )
endif()
