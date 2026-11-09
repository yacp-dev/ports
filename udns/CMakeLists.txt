cmake_minimum_required(VERSION 3.14)

# Take the version from udns.h so that it follows upstream on every bump.
file(STRINGS udns.h _udns_version_line REGEX "^#define UDNS_VERSION \"")
string(REGEX REPLACE ".*\"([0-9.]+)\".*" "\\1" UDNS_VERSION "${_udns_version_line}")

project(udns
    VERSION ${UDNS_VERSION}
    DESCRIPTION "DNS resolver library"
    LANGUAGES C
)

include(GNUInstallDirs)
include(CheckSymbolExists)
include(CheckCSourceCompiles)

# Same checks as upstream's configure.
check_symbol_exists(getopt unistd.h HAVE_GETOPT)
check_symbol_exists(inet_pton "sys/types.h;sys/socket.h;arpa/inet.h" HAVE_INET_PTON_NTOP)
check_symbol_exists(poll sys/poll.h HAVE_POLL)
check_c_source_compiles("
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
int main(void) { struct sockaddr_in6 sa; sa.sin6_family = AF_INET6; return 0; }
" HAVE_IPv6)

set(UDNS_DEFINITIONS "")
foreach(_def GETOPT INET_PTON_NTOP POLL IPv6)
    if(HAVE_${_def})
        list(APPEND UDNS_DEFINITIONS HAVE_${_def})
    endif()
endforeach()

set(UDNS_SOURCES
    udns_dn.c udns_dntosp.c udns_parse.c udns_resolver.c udns_init.c
    udns_misc.c udns_XtoX.c
    udns_rr_a.c udns_rr_ptr.c udns_rr_mx.c udns_rr_txt.c udns_bl.c
    udns_rr_srv.c udns_rr_naptr.c udns_codes.c udns_jran.c
)

add_library(udns SHARED ${UDNS_SOURCES})
target_include_directories(udns PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
target_compile_definitions(udns PRIVATE ${UDNS_DEFINITIONS})
# upstream's SOVER in Makefile.in
set_target_properties(udns PROPERTIES SOVERSION 0)

foreach(_prog dnsget rblcheck ex-rdns)
    add_executable(${_prog} ${_prog}.c)
    target_compile_definitions(${_prog} PRIVATE ${UDNS_DEFINITIONS})
    target_link_libraries(${_prog} PRIVATE udns)
endforeach()

install(TARGETS udns dnsget rblcheck ex-rdns
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
install(FILES udns.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES dnsget.1 rblcheck.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)
install(FILES udns.3 DESTINATION ${CMAKE_INSTALL_MANDIR}/man3)

configure_file(udns.pc.in udns.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/udns.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)
