cmake_minimum_required(VERSION 3.16)
project(esda_diff_drive_controller LANGUAGES CXX)

if(CMAKE_CXX_COMPILER_ID MATCHES "(GNU|Clang)")
  add_compile_options(-Wall -Wextra -Wconversion)
endif()

set(THIS_PACKAGE_INCLUDE_DEPENDS
  controller_interface
  generate_parameter_library
  geometry_msgs
  hardware_interface
  nav_msgs
  pluginlib
  rclcpp
  rclcpp_lifecycle
  rcpputils
  realtime_tools
  tf2
  tf2_msgs
  mavros_msgs
)

find_package(ament_cmake REQUIRED)
find_package(backward_ros REQUIRED)
foreach(Dependency IN ITEMS ${THIS_PACKAGE_INCLUDE_DEPENDS})
  find_package(${Dependency} REQUIRED)
endforeach()

generate_parameter_library(esda_diff_drive_controller_parameters
  src/esda_diff_drive_controller_parameter.yaml
)

add_library(esda_diff_drive_controller SHARED
  src/esda_diff_drive_controller.cpp
  src/odometry.cpp
  src/speed_limiter.cpp
)
target_compile_features(esda_diff_drive_controller PUBLIC cxx_std_17)
target_include_directories(esda_diff_drive_controller PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include/esda_diff_drive_controller>
)
target_link_libraries(esda_diff_drive_controller PUBLIC esda_diff_drive_controller_parameters)
ament_target_dependencies(esda_diff_drive_controller PUBLIC ${THIS_PACKAGE_INCLUDE_DEPENDS})
# Causes the visibility macros to use dllexport rather than dllimport,
# which is appropriate when building the dll but not consuming it.
target_compile_definitions(esda_diff_drive_controller PRIVATE "esda_diff_drive_controller_BUILDING_DLL")
pluginlib_export_plugin_description_file(controller_interface diff_drive_plugin.xml)

install(
  DIRECTORY include/
  DESTINATION include/esda_diff_drive_controller
)
install(TARGETS esda_diff_drive_controller esda_diff_drive_controller_parameters
  EXPORT export_esda_diff_drive_controller
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
)

ament_export_targets(export_esda_diff_drive_controller HAS_LIBRARY_TARGET)
ament_export_dependencies(${THIS_PACKAGE_INCLUDE_DEPENDS})
ament_package()
