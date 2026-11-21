cmake_minimum_required(VERSION 3.5...3.30)
project(cqrlib VERSION 1.1.4 LANGUAGES C CXX)

include(GNUInstallDirs)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

add_library(CQRlib SHARED cqrlib.c)
set_target_properties(CQRlib PROPERTIES
    OUTPUT_NAME CQRlib
    SOVERSION 2
    VERSION 2.0.1
)

install(TARGETS CQRlib
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(FILES cqrlib.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir ${CMAKE_INSTALL_PREFIX}/lib)
set(includedir ${CMAKE_INSTALL_PREFIX}/include)
set(VERSION ${PROJECT_VERSION})
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cqrlib.pc.in ${CMAKE_CURRENT_BINARY_DIR}/cqrlib.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cqrlib.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

enable_testing()
add_executable(CQRlibTest CQRlibTest.c)
target_link_libraries(CQRlibTest PRIVATE CQRlib)
add_test(NAME CQRlibTest COMMAND CQRlibTest)

add_executable(CPPQRTest CPPQRTest.cpp)
target_link_libraries(CPPQRTest PRIVATE CQRlib)
add_test(NAME CPPQRTest COMMAND CPPQRTest)
