cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(i3_my)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING
        "CMake build type not specified - default type Release chosen."
        FORCE)
endif(NOT CMAKE_BUILD_TYPE)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules/")

find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
endif(CCACHE_FOUND)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

find_package(PkgConfig REQUIRED)
pkg_search_module(PKG_I3IPC REQUIRED IMPORTED_TARGET i3ipc-glib-1.0)
pkg_search_module(PKG_X11 REQUIRED IMPORTED_TARGET x11)
pkg_search_module(PKG_XTST REQUIRED IMPORTED_TARGET xtst)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

find_package(Boost COMPONENTS program_options REQUIRED)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

add_custom_command(
  COMMAND ${CMAKE_COMMAND} -E create_symlink ${PROJECT_SOURCE_DIR} "${PROJECT_BINARY_DIR}/SOURCES"
  COMMAND ${CMAKE_COMMAND} -E touch custom_cmds.stamp
  OUTPUT custom_cmds.stamp
)
add_custom_target(custom_cmds ALL DEPENDS custom_cmds.stamp)

set (CMAKE_CXX_STANDARD 17)
set (CMAKE_CXX_STANDARD_REQUIRED ON)

set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wfatal-errors -Wshadow")
set (CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3")
set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")
message("Compiler ${CMAKE_CXX_COMPILER_ID}")
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
#  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
#  set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -stdlib=libc++")
#  set (CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS} -stdlib=libc++")
  # memory sanitizer
#  set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=memory -fPIE -fno-omit-frame-pointer -fsanitize-recover=all")
#  set (CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} -fsanitize=memory -fPIE -pie -fno-omit-frame-pointer")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  message("Applying settings for GCC")
  #set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  set (CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native -fexpensive-optimizations -finline-functions -fomit-frame-pointer")
  #set (CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -fprofile-generate")
  #set (CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -fprofile-use")
  # thread sanitizer
  #set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=thread")
  #set (CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} -fsanitize=thread")
  # address sanitizer
  #set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=address -fno-omit-frame-pointer")
  #set (CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} -fsanitize=address -fno-omit-frame-pointer")
  set (CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS} -flto")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_subdirectory(src)
