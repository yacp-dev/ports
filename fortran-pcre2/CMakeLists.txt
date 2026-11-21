cmake_minimum_required(VERSION 3.12)
project(fortran-pcre2 VERSION 0.1.1 LANGUAGES Fortran)

# Find pcre2-8 using pkg-config
find_package(PkgConfig REQUIRED)
pkg_check_modules(PCRE2 REQUIRED libpcre2-8)

# Target library
add_library(fortran-pcre2 SHARED src/pcre2.f90)
set_target_properties(fortran-pcre2 PROPERTIES SOVERSION ${PROJECT_VERSION_MAJOR})

# Link with pcre2-8
target_include_directories(fortran-pcre2 PRIVATE ${PCRE2_INCLUDE_DIRS})
target_link_libraries(fortran-pcre2 ${PCRE2_LIBRARIES})
target_link_directories(fortran-pcre2 PRIVATE ${PCRE2_LIBRARY_DIRS})

# Generate pkg-config file
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/fortran-pcre2.pc.in
    ${CMAKE_CURRENT_BINARY_DIR}/fortran-pcre2.pc
    @ONLY
)

# Installation rules
install(TARGETS fortran-pcre2 RUNTIME DESTINATION bin ARCHIVE DESTINATION lib)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/pcre2.mod 
        DESTINATION include/libfortran-pcre2)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/fortran-pcre2.pc
        DESTINATION lib/pkgconfig)

# Test executable
add_executable(test_pcre2 test/test_pcre2.f90)
target_link_libraries(test_pcre2 fortran-pcre2 ${PCRE2_LIBRARIES})

# Test target
enable_testing()
add_test(NAME test_pcre2 COMMAND test_pcre2)
