cmake_minimum_required(VERSION 3.16)
project(fibril_mujoco_vendor VERSION 3.11.0)

find_package(ament_cmake REQUIRED)

include(ExternalProject)

set(
  FIBRIL_MUJOCO_VERSION
  "3.11.0"
  CACHE STRING "MuJoCo release to build"
)
set(
  FIBRIL_MUJOCO_SOURCE_SHA256
  "f6346a0bab22bc0db5cabfe299fd3819b8b9bab67907c1ec0d243c675635ea3d"
  CACHE STRING "SHA256 of the source archive (update together with the version)"
)
set(FIBRIL_MUJOCO_SOURCE_URL
  "https://github.com/google-deepmind/mujoco/archive/refs/tags/${FIBRIL_MUJOCO_VERSION}.tar.gz"
)
set(FIBRIL_MUJOCO_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/mujoco_install")

if(CMAKE_BUILD_TYPE)
  set(FIBRIL_MUJOCO_BUILD_TYPE "${CMAKE_BUILD_TYPE}")
else()
  set(FIBRIL_MUJOCO_BUILD_TYPE "Release")
endif()

ExternalProject_Add(
  mujoco_external
  URL "${FIBRIL_MUJOCO_SOURCE_URL}"
  URL_HASH "SHA256=${FIBRIL_MUJOCO_SOURCE_SHA256}"
  DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  CMAKE_ARGS
    "-DCMAKE_BUILD_TYPE=${FIBRIL_MUJOCO_BUILD_TYPE}"
    "-DCMAKE_INSTALL_PREFIX=${FIBRIL_MUJOCO_INSTALL_DIR}"
    "-DCMAKE_SHARED_LINKER_FLAGS=-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/cmake/mujoco.exports.map"
    "-DMUJOCO_ENABLE_LTO=OFF"
    "-DMUJOCO_BUILD_EXAMPLES=OFF"
    "-DMUJOCO_BUILD_SIMULATE=OFF"
    "-DMUJOCO_BUILD_STUDIO=OFF"
    "-DMUJOCO_BUILD_TESTS=OFF"
    "-DMUJOCO_TEST_PYTHON_UTIL=OFF"
    "-DMUJOCO_WITH_USD=OFF"
    "-DMUJOCO_USE_FILAMENT=OFF"
)

install(
  DIRECTORY "${FIBRIL_MUJOCO_INSTALL_DIR}/"
  DESTINATION "opt/${PROJECT_NAME}"
)

# ament が既定で作る environment/library_path.dsv (中身は lib) と名前がぶつかると上書きされるので別名にする
ament_environment_hooks(environment/fibril_mujoco_vendor_library_path.dsv)

configure_file(
  cmake/fibril_mujoco_vendor-extras.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/fibril_mujoco_vendor-extras.cmake"
  @ONLY
)

ament_package(
  CONFIG_EXTRAS "${CMAKE_CURRENT_BINARY_DIR}/fibril_mujoco_vendor-extras.cmake"
)
