cmake_minimum_required(VERSION 3.10)
project(jsmn C)

# Set project version
set(PROJECT_VERSION 0)

# Source files
set(JSMN_SOURCES jsmn.c)
set(JSMN_HEADERS jsmn.h)

# Create static library
add_library(jsmn SHARED ${JSMN_SOURCES})
target_include_directories(jsmn PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(jsmn PROPERTIES SOVERSION 0)

# Example: simple_example executable
add_executable(simple_example example/simple.c)
target_link_libraries(simple_example PRIVATE jsmn)

# Example: jsondump executable
add_executable(jsondump example/jsondump.c)
target_link_libraries(jsondump PRIVATE jsmn)

# Test executables
enable_testing()

# test_default
add_executable(test_default test/tests.c)
target_link_libraries(test_default PRIVATE jsmn)
add_test(NAME test_default COMMAND test_default)

# test_strict
add_executable(test_strict test/tests.c)
target_compile_definitions(test_strict PRIVATE JSMN_STRICT=1)
target_link_libraries(test_strict PRIVATE jsmn)
add_test(NAME test_strict COMMAND test_strict)

# test_links
add_executable(test_links test/tests.c)
target_compile_definitions(test_links PRIVATE JSMN_PARENT_LINKS=1)
target_link_libraries(test_links PRIVATE jsmn)
add_test(NAME test_links COMMAND test_links)

# test_strict_links
add_executable(test_strict_links test/tests.c)
target_compile_definitions(test_strict_links PRIVATE JSMN_STRICT=1 JSMN_PARENT_LINKS=1)
target_link_libraries(test_strict_links PRIVATE jsmn)
add_test(NAME test_strict_links COMMAND test_strict_links)

# Custom test target
add_custom_target(test_all COMMAND ${CMAKE_CTEST_COMMAND})

# Generate pkg-config file
configure_file(jsmn.pc.in jsmn.pc @ONLY)

# Install targets
install(TARGETS jsmn RUNTIME DESTINATION bin ARCHIVE DESTINATION lib)
install(FILES ${JSMN_HEADERS} DESTINATION include)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/jsmn.pc DESTINATION lib/pkgconfig)
