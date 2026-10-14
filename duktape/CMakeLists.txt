cmake_minimum_required(VERSION 3.28)
project(duktape
  VERSION 2.7.0
  DESCRIPTION "Embeddable Javascript engine"
  HOMEPAGE_URL "https://duktape.org/")

include(GNUInstallDirs)

add_library(duktape SHARED src/duktape.c)
set_target_properties(duktape PROPERTIES SOVERSION 207)

find_package(pkgconfig)
pkg_check_modules(LINENOISE linenoise)

add_executable(duk examples/cmdline/duk_cmdline.c
  extras/print-alert/duk_print_alert.c
  extras/console/duk_console.c
  extras/logging/duk_logging.c
  extras/module-duktape/duk_module_duktape.c)
target_include_directories(duk PRIVATE examples/cmdline src
  extras/print-alert
  extras/console
  extras/logging
  extras/module-duktape
  ${LINENOISE_INCLUDE_DIRS})
target_compile_definitions(duk PRIVATE
  -DDUK_CMDLINE_PRINTALERT_SUPPORT
  -DDUK_CMDLINE_CONSOLE_SUPPORT
  -DDUK_CMDLINE_LOGGING_SUPPORT
  -DDUK_CMDLINE_MODULE_SUPPORT
  -DDUK_CMDLINE_FANCY)
target_link_libraries(duk duktape ${LINENOISE_LIBRARIES})

install(TARGETS duk
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(TARGETS duktape
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES src/duk_config.h src/duktape.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(VERSION ${PROJECT_VERSION})
set(PREFIX ${CMAKE_INSTALL_PREFIX})
set(LIBDIR /lib)
configure_file(${CMAKE_SOURCE_DIR}/duktape.pc.in ${CMAKE_BINARY_DIR}/duktape.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/duktape.pc DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig)
