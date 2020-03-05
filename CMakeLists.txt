CMAKE_MINIMUM_REQUIRED(VERSION 3.10)
PROJECT(nfd VERSION 0.9)

# project root directory
SET(NFD_ROOT ${PROJECT_SOURCE_DIR}/nfd)

# project include directories
INCLUDE_DIRECTORIES(${NFD_ROOT}/include)
INCLUDE_DIRECTORIES(${NFD_ROOT}/include/public)

# set source and header file sources
SET(NFD_SRC_DIR ${NFD_ROOT}/src)
SET(NFD_SRC 
      "${NFD_SRC_DIR}/nfd_common.c"
   ) # source files
SET(NFD_EXPOSE_HEADERS 
   "${NFD_ROOT}/include/public"
) # exposed library header files

# set link libraries to be defined per each platform later.
SET(NFD_DEPS )



IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
   # Mac OS X specific code
   SET(OS "Mac OS X")
   message("compiling code for ${OS}")
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")


IF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

   # Linux specific code
   SET(OS "Linux")
   MESSAGE("compiling native file dialog library for ${OS}")

   FIND_PACKAGE(PkgConfig REQUIRED)
   PKG_CHECK_MODULES(GTK3 REQUIRED gtk+-3.0)
   
   # find and link against gtk3 libraries
   INCLUDE_DIRECTORIES(${GTK3_INCLUDE_DIRS})
   LINK_DIRECTORIES(${GTK3_LIBRARY_DIRS})
   
   # add other flags to the compiler
   ADD_DEFINITIONS(${GTK3_CFLAGS_OTHER})

   # append linux source files to the NFD_SRC var
   SET(NFD_SRC ${NFD_SRC_DIR}/nfd_gtk.c ${NFD_SRC})
   MESSAGE("source files : ${NFD_SRC}")
   
   # append linux dependencies to the list 
   SET(NFD_DEPS ${GTK3_LIBRARIES} ${NFD_DEPS})

   ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
   
   
   # add library and 
   ADD_LIBRARY(lib_nfd SHARED ${NFD_SRC})
   TARGET_LINK_LIBRARIES(lib_nfd ${NFD_DEPS})
   target_include_directories(lib_nfd PUBLIC ${NFD_EXPOSE_HEADERS})
   
   
   # test working nfd lib 
   OPTION(GENERATE_TESTS "CHECK WORKING NFD LIB" OFF)
   IF(GENERATE_TESTS)
      MESSAGE("GENERATING TEST EXEC")
      ADD_EXECUTABLE(nfd_test_open ${NFD_ROOT}/test/test_opendialog.c)
      TARGET_LINK_LIBRARIES(nfd_test_open lib_nfd)
   ENDIF(GENERATE_TESTS)
   UNSET(GENERATE_TESTS CACHE)