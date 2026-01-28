cmake_minimum_required(VERSION 3.20)

project(proton_vendor)

# Find ROS 2 dependencies
find_package(ament_cmake REQUIRED)

# Fetch Proton
include(FetchContent)
FetchContent_Declare(
  proton
  GIT_REPOSITORY https://github.com/clearpathrobotics/proton.git
  GIT_TAG main
)

set(PROTON_BUILD_EXAMPLES OFF)
set(PROTON_INSTALL ON)

FetchContent_MakeAvailable(proton)

# Install proton libraries and headers
install(
  TARGETS protonc protoncpp
  EXPORT proton_vendor_targets
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# Install proton headers
install(
  DIRECTORY ${proton_SOURCE_DIR}/include/
  DESTINATION include
)

install(
  DIRECTORY ${proton_SOURCE_DIR}/protonc/include/
  DESTINATION include
)

install(
  DIRECTORY ${proton_SOURCE_DIR}/protoncpp/include/
  DESTINATION include
)

# Install generated headers from the build directory
install(
  DIRECTORY ${proton_BINARY_DIR}/protonc/generated/
  DESTINATION include/protonc
  FILES_MATCHING PATTERN "*.h"
)

install(
  DIRECTORY ${proton_BINARY_DIR}/protoncpp/generated/
  DESTINATION include
  FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
)

# Export targets
install(
  EXPORT proton_vendor_targets
  FILE proton_vendor_targets.cmake
  NAMESPACE proton_vendor::
  DESTINATION share/${PROJECT_NAME}/cmake
)

# Export dependencies
ament_export_targets(proton_vendor_targets HAS_LIBRARY_TARGET)
ament_export_dependencies()

ament_package()
