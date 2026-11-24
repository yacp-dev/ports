cmake_minimum_required(VERSION 3.10)
project(mongoose C)

# The version is defined in mongoose.h only
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/mongoose.h" MG_VERSION_LINE
     REGEX "^#define MG_VERSION \"[0-9.]+\"")
string(REGEX REPLACE ".*\"([0-9]+)\\.([0-9]+)(\\.([0-9]+))?\".*" "\\1;\\2"
       MG_VERSION_PARTS "${MG_VERSION_LINE}")
list(GET MG_VERSION_PARTS 0 MG_VERSION_MAJOR)
list(GET MG_VERSION_PARTS 1 MG_VERSION_MINOR)
string(REGEX REPLACE ".*\"([0-9.]+)\".*" "\\1" MG_VERSION "${MG_VERSION_LINE}")

include(GNUInstallDirs)
include(CTest)
find_package(OpenSSL REQUIRED)
find_package(Threads REQUIRED)

set(MG_DEFINITIONS
    MG_TLS=MG_TLS_OPENSSL
    MG_ENABLE_LINES=1
    MG_ENABLE_DIRLIST=1
    MG_ENABLE_SSI=1
    MG_ENABLE_IPV6=1)

add_library(mongoose SHARED mongoose.c)
target_compile_definitions(mongoose PRIVATE ${MG_DEFINITIONS})
target_include_directories(mongoose PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/mongoose>)
target_link_libraries(mongoose PRIVATE OpenSSL::SSL OpenSSL::Crypto)
# Upstream has no stable ABI: the runtime is named after the release
# (cygmongoose-7-24.dll), the import library stays libmongoose.dll.a
set_target_properties(mongoose PROPERTIES
    RUNTIME_OUTPUT_NAME "mongoose-${MG_VERSION_MAJOR}-${MG_VERSION_MINOR}"
    ARCHIVE_OUTPUT_NAME mongoose)

install(TARGETS mongoose
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES mongoose.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/mongoose)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/mongoose.pc"
"prefix=${CMAKE_INSTALL_PREFIX}
exec_prefix=\${prefix}
libdir=${CMAKE_INSTALL_FULL_LIBDIR}
includedir=${CMAKE_INSTALL_FULL_INCLUDEDIR}

Name: mongoose
Description: Embedded web server for C/C++
Version: ${MG_VERSION}
Libs: -L\${libdir} -lmongoose
Cflags: -I\${includedir}/mongoose
Requires: libcrypto libssl
")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/mongoose.pc"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

if(BUILD_TESTING)
  # The unit test embeds test files into packed_fs.c with its own tool
  set(TEST_DIR "${CMAKE_CURRENT_SOURCE_DIR}/test")
  add_executable(mongoose_pack "${TEST_DIR}/pack.c")
  set(PACKED_FILES Makefile data/ssi.h fuzz.c data/a.txt data/range.txt
      data/ca.pem certs/ca.crt certs/server.crt certs/server.key
      certs/client.key certs/client.crt)
  add_custom_command(
      OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/packed_fs.c"
      COMMAND mongoose_pack ${PACKED_FILES}
          > "${CMAKE_CURRENT_BINARY_DIR}/packed_fs.c"
      WORKING_DIRECTORY "${TEST_DIR}"
      DEPENDS mongoose_pack
      VERBATIM)
  add_executable(unit_test "${TEST_DIR}/unit_test.c"
      "${CMAKE_CURRENT_BINARY_DIR}/packed_fs.c")
  target_compile_definitions(unit_test PRIVATE ${MG_DEFINITIONS})
  target_link_libraries(unit_test PRIVATE mongoose Threads::Threads)
  add_test(NAME unit_test COMMAND unit_test WORKING_DIRECTORY "${TEST_DIR}")
endif()
