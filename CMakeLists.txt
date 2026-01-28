cmake_minimum_required(VERSION 3.20)

project(proton_vendor)

# Find ROS 2 dependencies
find_package(ament_cmake REQUIRED)

include(ExternalProject)

set(PROTON_EXTERNAL_PROJECT_NAME proton_external_project)
set(PROTON_GIT_REPOSITORY "https://github.com/clearpathrobotics/proton.git")
set(PROTON_GIT_TAG "main")

set(EXTERNAL_PROJECT_PREFIX ${CMAKE_CURRENT_BINARY_DIR}/proton_external_build)
set(EXTERNAL_PROJECT_SOURCE_DIR ${EXTERNAL_PROJECT_PREFIX}/src/proton)
set(EXTERNAL_PROJECT_BINARY_DIR ${EXTERNAL_PROJECT_PREFIX}/build/proton)
set(EXTERNAL_PROJECT_INSTALL_DIR ${CMAKE_INSTALL_PREFIX}) # Proton library should install its files (libraries, headers) into proton_vendor package install space.

# Use sequential build for ExternalProject to avoid file descriptor issues
set(EXTERNAL_PROJECT_BUILD_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --parallel 1)

ExternalProject_Add(${PROTON_EXTERNAL_PROJECT_NAME}
    # download step
    GIT_REPOSITORY    ${PROTON_GIT_REPOSITORY}
    GIT_TAG           ${PROTON_GIT_TAG}

    # directiories config
    PREFIX            ${EXTERNAL_PROJECT_PREFIX}
    SOURCE_DIR        ${EXTERNAL_PROJECT_SOURCE_DIR}
    BINARY_DIR        ${EXTERNAL_PROJECT_BINARY_DIR}
    INSTALL_DIR       ${EXTERNAL_PROJECT_INSTALL_DIR}

    # after initial download, don't re-fetch source on subsequent CMake runs
    UPDATE_COMMAND    ""

    # configure step
    CONFIGURE_COMMAND ${CMAKE_COMMAND} -S <SOURCE_DIR> -B <BINARY_DIR>
                      -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
                      -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
                      -DCMAKE_CROSSCOMPILING=OFF
                      -DBUILD_SHARED_LIBS=ON
                      -DPROTON_BUILD_PROTONC=ON
                      -DPROTON_BUILD_PROTONCPP=ON
                      -DPROTON_BUILD_EXAMPLES=OFF
                      -DPROTON_BUILD_TESTS=OFF
                      -DPROTON_INSTALL=ON

    # build step
    BUILD_COMMAND     ${EXTERNAL_PROJECT_BUILD_COMMAND}

    # install step
    INSTALL_COMMAND   ""

    # logging
    LOG_CONFIGURE     YES
    LOG_BUILD         YES
    LOG_INSTALL       YES
)

# ros buildfarm fix: separate ExternalProject install step from the clone/build steps.
# Reference: https://github.com/ros-industrial/ros2_canopen/blob/master/lely_core_libraries/CMakeLists.txt
install(CODE "execute_process(COMMAND ${CMAKE_COMMAND} --install ${EXTERNAL_PROJECT_BINARY_DIR})")

add_library(proton_vendor INTERFACE)
add_library(proton_vendor::proton_vendor ALIAS proton_vendor)

target_include_directories(proton_vendor
  INTERFACE
  $<INSTALL_INTERFACE:include>
)
target_link_libraries(proton_vendor
  INTERFACE
  ${CMAKE_INSTALL_PREFIX}/lib/libprotonc.so
  ${CMAKE_INSTALL_PREFIX}/lib/libprotoncpp.so
)

install(
  TARGETS proton_vendor
  EXPORT proton_vendor_targets
)

ament_export_targets(proton_vendor_targets HAS_LIBRARY_TARGET)

ament_package()
