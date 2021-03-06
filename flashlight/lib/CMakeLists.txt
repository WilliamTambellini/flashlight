cmake_minimum_required(VERSION 3.10)

# ------------------------- Library -------------------------

add_library(
  fl-libraries
  ""
  )
set(INSTALLABLE_TARGETS ${INSTALLABLE_TARGETS} fl-libraries)

target_include_directories(
  fl-libraries
  PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
  )

target_compile_definitions(
  fl-libraries
  PUBLIC
  ${FL_COMPILE_DEFINITIONS}
  )

# --------------------------- Components ---------------------------

# sequence
include(${CMAKE_CURRENT_LIST_DIR}/sequence/CMakeLists.txt)

# audio
include(${CMAKE_CURRENT_LIST_DIR}/audio/CMakeLists.txt)

# common
include(${CMAKE_CURRENT_LIST_DIR}/common/CMakeLists.txt)

# text
include(${CMAKE_CURRENT_LIST_DIR}/text/CMakeLists.txt)

# ---------------------- CUDA Library -----------------------

# Only build the library if the FL CUDA backend is enabled.
if (NOT FL_USE_CUDA)
  message(STATUS "FL_BACKEND is not CUDA - forcing FL_LIBRARIES_USE_CUDA to OFF")
  set(FL_LIBRARIES_USE_CUDA OFF)
endif()

# Accumulate CUDA sources after including library components because
# target_sources doesn't work with CUDA libraries as expected with
# old versions of CMake
set(FL_LIBRARY_CUDA_SOURCES
  ${FL_LIBRARIES_SEQUENCE_CUDA_SOURCES}
  )

# A library containing CUDA targets within libraries that may need to be linked
# in. Only build a CUDA lib of libraries that require CUDA kernels are built
if (FL_LIBRARIES_USE_CUDA)
  # Qualify name because CUDA target is public
  target_sources(
    fl-libraries
    PRIVATE
    ${FL_LIBRARY_CUDA_SOURCES}
    )

  # CUB
  if(CUDA_VERSION_MAJOR VERSION_LESS 11)
    find_path(CUB_INCLUDE_DIRS "cub/cub.cuh")
    if (NOT ${CUB_INCLUDE_DIRS})
      include(${CMAKE_MODULE_PATH}/BuildCUB.cmake)
      add_dependencies(fl-libraries CUB)
    endif()
  endif()

  target_include_directories(
    fl-libraries
    PUBLIC
    $<BUILD_INTERFACE:${CUB_INCLUDE_DIRS}>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CUDA_INCLUDE_DIRS}>
    )

  target_link_libraries(
    fl-libraries
    PUBLIC
    ${CUDA_LIBRARIES}
    )
endif()

# --------------------------- Tests ---------------------------

if (FL_BUILD_TESTS)
  add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/test)
endif ()
