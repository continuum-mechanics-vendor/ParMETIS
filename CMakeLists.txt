cmake_minimum_required(VERSION 3.26)

if(NOT CMAKE_BUILD_TYPE AND NOT DEFINED ENV{CMAKE_BUILD_TYPE})
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Debug or Release")
endif()

project(PARMETIS LANGUAGES C VERSION 1.0.0)

option(PARMETIS_BUILD_PROGRAMS "Build ParMETIS programs" OFF)

# --- need METIS first
include(GNUInstallDirs)

set(CMAKE_C_STANDARD 99)

add_compile_definitions(
  "REALTYPEWIDTH=$<IF:$<BOOL:${REALTYPEWIDTH}>,${REALTYPEWIDTH},32>"
  "IDXTYPEWIDTH=$<IF:$<BOOL:${IDXTYPEWIDTH}>,${IDXTYPEWIDTH},32>"
)

find_package(METIS REQUIRED)
find_package(GKlib REQUIRED)
find_package(MPI REQUIRED)

add_library(parmetis)
target_include_directories(parmetis PUBLIC
  "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include;${PROJECT_SOURCE_DIR}/libparmetis>"
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
target_link_libraries(parmetis PUBLIC MPI::MPI_C METIS::METIS GKlib::GKlib)

add_library(PARMETIS::PARMETIS ALIAS parmetis)

install(TARGETS parmetis EXPORT ${PROJECT_NAME}-targets)

add_subdirectory(libparmetis)

if(PARMETIS_BUILD_PROGRAMS)
  add_subdirectory(programs)
endif()

# This is for testing during development and is not being distributed
#add_subdirectory(test)

include(cmake/install.cmake)

file(GENERATE OUTPUT .gitignore CONTENT "*")

