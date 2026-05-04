# https://github.com/CharlesGueunet/kakoune/blob/master/CMakeLists.txt

cmake_minimum_required(VERSION 3.11)
project(Kakoune LANGUAGES CXX)

# Prerequisites
set(CMAKE_CXX_STANDARD 20)

# Kakoune version
option(MANUAL_VERSION "Use a manually defined version instead of the date" FALSE)
mark_as_advanced(MANUAL_VERSION)
if(${MANUAL_VERSION})
  set(KAKOUNE_VERSION "0.0" CACHE STRING "Kakoune version")
else()
  string(TIMESTAMP KAKOUNE_VERSION "%Y.%m.%d")
endif()
project(Kakoune VERSION ${KAKOUNE_VERSION})

# Configuration variables
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to 'Release'.")
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release")
endif()
option(BUILD_SHARED_LIBS "Use static or shared libs" TRUE)

# Compile files and main target
add_executable(kak)
file(GLOB KAKOUNE_SOURCES src/*.cc)

# speial case of the version.cc.in file
configure_file(src/version.cc.in src/version.cc)
list(APPEND KAKOUNE_SOURCES ${CMAKE_CURRENT_BINARY_DIR}/src/version.cc)
# add cc files
target_sources(kak PRIVATE ${KAKOUNE_SOURCES})

# Compile flags for kak
if(CMAKE_BUILD_TYPE MATCHES Debug)
  target_compile_definitions(kak PRIVATE KAK_DEBUG)
endif()
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  target_compile_options(kak PRIVATE "-Wno-init-list-lifetime")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  target_compile_options(kak PRIVATE "-frelaxed-template-template-args")
endif()

# Install binary and files
install(TARGETS kak
        RUNTIME DESTINATION bin)
# share folder
install(FILES share/kak/kakrc DESTINATION share/kak)
install(DIRECTORY colors/     DESTINATION share/kak/colors/)
install(DIRECTORY rc/         DESTINATION share/kak/rc/)
# share/kak/autoload/ is symlink to rc/
install(CODE "execute_process(COMMAND ${CMAKE_COMMAND} -E \
                              create_symlink rc/ \${CMAKE_INSTALL_PREFIX}/share/kak/autoload)"
)
# (TODO need to compress)
install(FILES doc/kak.1       DESTINATION share/man/man1/)
install(FILES README.asciidoc DESTINATION share/doc/kak/)
install(DIRECTORY doc/pages/  DESTINATION share/kak/doc/)

# testing
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/test/run ${CMAKE_CURRENT_BINARY_DIR}/run COPYONLY)
enable_testing()
include(CTest)
add_test(NAME kak_test
         COMMAND run ${CMAKE_CURRENT_BINARY_DIR}/kak)
