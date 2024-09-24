cmake_minimum_required(VERSION 3.8)
project(robotiq_driver LANGUAGES CXX)

# This module provides installation directories as per the GNU coding standards.
include(GNUInstallDirs)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)
find_package(serial REQUIRED)  # Ensure serial is found

# Robotiq driver library.

add_library(
  robotiq_driver
  SHARED
  src/crc_utils.cpp
  src/data_utils.cpp
  src/hardware_interface.cpp
  src/default_driver.cpp
  src/default_driver_factory.cpp
  src/fake/fake_driver.cpp
  src/default_serial.cpp
  src/default_serial_factory.cpp
)

# Link against the serial library
target_link_libraries(robotiq_driver ${serial_LIBRARIES})

# Include directories
target_include_directories(
  robotiq_driver
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  ${serial_INCLUDE_DIRS}  # Add serial include directories
)

ament_target_dependencies(
  robotiq_driver
  hardware_interface
  pluginlib
  rclcpp
  rclcpp_lifecycle
  serial
)

###############################################################################
# PLUGINS

pluginlib_export_plugin_description_file(hardware_interface hardware_interface_plugin.xml)

###############################################################################
# EXPORTS

ament_export_targets(
  robotiq_driver_targets # Must match the EXPORT label below in the install section.
)
ament_export_dependencies(
  hardware_interface
  pluginlib
  rclcpp
  rclcpp_lifecycle
  serial
)
ament_export_include_directories(
  include
)
ament_export_libraries(
  robotiq_driver
)

###############################################################################
# INSTALL

install(
    DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
  TARGETS robotiq_driver
  EXPORT robotiq_driver_targets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

###############################################################################
# TESTS

include(CTest)
if(BUILD_TESTING)
  add_subdirectory(tests)
endif()

###############################################################################
# LINTERS

add_custom_target(format
    COMMAND clang-format -i `git ls-files *.hpp *.cpp`
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
add_custom_target(tidy
    COMMAND clang-tidy -p ${CMAKE_BINARY_DIR} `git ls-files *.cpp`
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})

ament_package()
