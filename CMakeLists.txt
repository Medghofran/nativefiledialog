cmake_minimum_required(VERSION 3.10)
project(nfd VERSION 0.9)

# project root directory
set(NFD_ROOT ${PROJECT_SOURCE_DIR}/nfd)

# project include directories
include_directories(${NFD_ROOT}/include)
include_directories(${NFD_ROOT}/include/public)

# set source and header file sources
set(NFD_SRC_DIR ${NFD_ROOT}/src)
set(NFD_SRC "${NFD_SRC_DIR}/nfd_common.c") # source files
set(NFD_EXPOSE_HEADERS "${NFD_ROOT}/include/public") # exposed library header
                                                     # files

# set link libraries to be defined per each platform later.
set(NFD_DEPS)

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  # Mac OS X specific code
  set(OS "Mac OS X")
  message(STATUS "Compiling NFD Library for target: ${OS}")

  # append linux source files to the NFD_SRC var
  set(NFD_SRC ${NFD_SRC_DIR}/nfd_cocoa.m ${NFD_SRC})

  # append linux dependencies to the list
  set(NFD_DEPS ${NFD_DEPS} "-framework AppKit" "-framework Foundation")
  set(NFD_DEPS ${NFD_DEPS} objc)

  # MAC OS SPECIFIC: set the linker flags to link against the foundation and
  # appkit frameworks
  set(CMAKE_EXE_LINKER_FLAGS
      "-bind_at_load -framework AppKit -framework Foundation ")

endif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

  # Linux specific code
  set(OS "Linux")
  message(STATUS "Compiling NFD Library for target:  ${OS}")

  find_package(PkgConfig REQUIRED)
  pkg_check_modules(GTK3 REQUIRED gtk+-3.0)

  # find and link against gtk3 libraries
  include_directories(${GTK3_INCLUDE_DIRS})
  link_directories(${GTK3_LIBRARY_DIRS})

  # add other flags to the compiler
  add_definitions(${GTK3_CFLAGS_OTHER})

  # append linux source files to the NFD_SRC var
  set(NFD_SRC ${NFD_SRC_DIR}/nfd_gtk.c ${NFD_SRC})

  # append linux dependencies to the list
  set(NFD_DEPS ${GTK3_LIBRARIES} ${NFD_DEPS})

endif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

# add library and
add_library(lib_nfd SHARED ${NFD_SRC})
target_link_libraries(lib_nfd PRIVATE ${NFD_DEPS})
target_include_directories(lib_nfd PUBLIC ${NFD_EXPOSE_HEADERS})

# Generate Test Executables for target
option(GENERATE_TESTS "CHECK WORKING NFD LIB" OFF)
if(GENERATE_TESTS)
  message(STATUS "GENERATING TEST EXECs")
  add_executable(nfd_test_open ${NFD_ROOT}/test/test_opendialog.c)
  target_link_libraries(nfd_test_open lib_nfd)
endif(GENERATE_TESTS)
unset(GENERATE_TESTS CACHE)
