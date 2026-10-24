cmake_minimum_required(VERSION 3.28)
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/linear.h" LIBLINEAR_VERSION REGEX "^#define LIBLINEAR_VERSION ")
string(REGEX REPLACE "^#define LIBLINEAR_VERSION[ \t]+([0-9]+)$" "\\1" LIBLINEAR_VERSION "${LIBLINEAR_VERSION}")
math(EXPR LIBLINEAR_VERSION_MAJOR "${LIBLINEAR_VERSION} / 100")
math(EXPR LIBLINEAR_VERSION_MINOR "${LIBLINEAR_VERSION} % 100")
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/Makefile" LIBLINEAR_SOVERSION REGEX "^SHVER = ")
string(REGEX REPLACE "^SHVER = ([0-9]+)$" "\\1" LIBLINEAR_SOVERSION "${LIBLINEAR_SOVERSION}")
project(liblinear
  VERSION ${LIBLINEAR_VERSION_MAJOR}.${LIBLINEAR_VERSION_MINOR}
  DESCRIPTION "Library for Large Linear Classification"
  HOMEPAGE_URL "https://www.csie.ntu.edu.tw/~cjlin/liblinear/")

include(GNUInstallDirs)
include(CTest)

find_package(BLAS REQUIRED)

add_library(linear SHARED linear.cpp newton.cpp)
target_link_libraries(linear ${BLAS_LIBRARIES})
set_target_properties(linear PROPERTIES SOVERSION ${LIBLINEAR_SOVERSION})
install(TARGETS linear
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

add_executable(predict predict.c)
target_link_libraries(predict linear ${BLAS_LIBRARIES})
install(TARGETS predict RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

add_executable(train train.c)
target_link_libraries(train linear ${BLAS_LIBRARIES})
install(TARGETS train RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(FILES linear.h newton.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/liblinear)

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir ${CMAKE_INSTALL_PREFIX}/lib)
set(includedir ${CMAKE_INSTALL_PREFIX}/include)
configure_file(${CMAKE_SOURCE_DIR}/liblinear.pc.in ${CMAKE_BINARY_DIR}/liblinear.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/liblinear.pc DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig)
