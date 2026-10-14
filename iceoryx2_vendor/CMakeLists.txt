cmake_minimum_required(VERSION 3.16)
project(iceoryx2_vendor VERSION 0.9.3)

find_package(ament_cmake REQUIRED)

include(ExternalProject)

set(
  FIBRIL_ICEORYX2_VERSION
  "0.9.3"
  CACHE STRING "iceoryx2 release to build"
)
set(
  FIBRIL_ICEORYX2_SOURCE_SHA256
  "2b4438364614b390938354ac766c198bb2ad63a2bde84a142c53f555fe8f4bc1"
  CACHE STRING "SHA256 of the source archive (update together with the version)"
)
set(
  FIBRIL_ICEORYX2_MINIMUM_RUST_VERSION
  "1.85.0"
  CACHE STRING "Minimum rustc version required by the selected iceoryx2 release"
)
set(FIBRIL_ICEORYX2_SOURCE_URL
  "https://github.com/eclipse-iceoryx/iceoryx2/archive/refs/tags/v${FIBRIL_ICEORYX2_VERSION}.tar.gz"
)
set(FIBRIL_ICEORYX2_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/iceoryx2_install")

find_program(FIBRIL_ICEORYX2_CARGO_EXECUTABLE cargo REQUIRED)
find_program(FIBRIL_ICEORYX2_RUSTC_EXECUTABLE rustc REQUIRED)

execute_process(
  COMMAND "${FIBRIL_ICEORYX2_RUSTC_EXECUTABLE}" --version
  OUTPUT_VARIABLE FIBRIL_ICEORYX2_RUSTC_VERSION_OUTPUT
  OUTPUT_STRIP_TRAILING_WHITESPACE
  COMMAND_ERROR_IS_FATAL ANY
)
string(
  REGEX MATCH "^rustc ([0-9]+\\.[0-9]+\\.[0-9]+)"
  FIBRIL_ICEORYX2_RUSTC_VERSION_MATCH
  "${FIBRIL_ICEORYX2_RUSTC_VERSION_OUTPUT}"
)
if(NOT FIBRIL_ICEORYX2_RUSTC_VERSION_MATCH)
  message(FATAL_ERROR
    "Unable to parse rustc version from: ${FIBRIL_ICEORYX2_RUSTC_VERSION_OUTPUT}"
  )
endif()
set(FIBRIL_ICEORYX2_RUSTC_VERSION "${CMAKE_MATCH_1}")
if(FIBRIL_ICEORYX2_RUSTC_VERSION VERSION_LESS FIBRIL_ICEORYX2_MINIMUM_RUST_VERSION)
  message(FATAL_ERROR
    "iceoryx2 ${FIBRIL_ICEORYX2_VERSION} requires rustc >= "
    "${FIBRIL_ICEORYX2_MINIMUM_RUST_VERSION}, found ${FIBRIL_ICEORYX2_RUSTC_VERSION}. "
    "Install a compatible Rust toolchain before building Fibril Simulation."
  )
endif()

if(CMAKE_BUILD_TYPE)
  set(FIBRIL_ICEORYX2_BUILD_TYPE "${CMAKE_BUILD_TYPE}")
else()
  set(FIBRIL_ICEORYX2_BUILD_TYPE "Release")
endif()

ExternalProject_Add(
  iceoryx2_external
  URL "${FIBRIL_ICEORYX2_SOURCE_URL}"
  URL_HASH "SHA256=${FIBRIL_ICEORYX2_SOURCE_SHA256}"
  DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  CMAKE_ARGS
    "-DCMAKE_BUILD_TYPE=${FIBRIL_ICEORYX2_BUILD_TYPE}"
    "-DCMAKE_INSTALL_PREFIX=${FIBRIL_ICEORYX2_INSTALL_DIR}"
    "-DBUILD_CXX=ON"
    "-DBUILD_EXAMPLES=OFF"
    "-DBUILD_TESTING=OFF"
)

install(
  DIRECTORY "${FIBRIL_ICEORYX2_INSTALL_DIR}/"
  DESTINATION "opt/${PROJECT_NAME}"
)

# ament が既定で作る environment/library_path.dsv (中身は lib) と名前がぶつかると上書きされるので別名にする
ament_environment_hooks(environment/iceoryx2_vendor_library_path.dsv)

configure_file(
  cmake/iceoryx2_vendor-extras.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/iceoryx2_vendor-extras.cmake"
  @ONLY
)

ament_package(
  CONFIG_EXTRAS "${CMAKE_CURRENT_BINARY_DIR}/iceoryx2_vendor-extras.cmake"
)
