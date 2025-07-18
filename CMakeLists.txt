cmake_minimum_required(VERSION 3.8)
project(conveyor_sorter)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(std_srvs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(ros_gz_bridge REQUIRED)
find_package(ros_gz_interfaces REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(OpenCV REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "srv/ConveyorStart.srv"
  "srv/SpawnProp.srv"
)
ament_export_dependencies(rosidl_default_runtime)

set(dependencies
  ${rclcpp_TARGETS}
  ${ament_index_cpp_TARGETS}
  ${std_msgs_TARGETS}
  ${std_srvs_TARGETS}
  ${sensor_msgs_TARGETS}
  ${ros_gz_bridge_TARGETS}
  ${ros_gz_interfaces_TARGETS}
)

add_executable(ros_gz_bridge src/ros_gz_bridge.cpp)
target_link_libraries(
  ros_gz_bridge
  PUBLIC ${dependencies}
)

add_executable(pusher_controller src/pusher_controller.cpp)
target_link_libraries(
  pusher_controller
  PUBLIC ${dependencies}
)

add_executable(conveyor_controller src/conveyor_controller.cpp)
target_link_libraries(
  conveyor_controller
  PUBLIC ${dependencies}
  "${PROJECT_NAME}__rosidl_typesupport_cpp"
)

add_executable(prop_spawner src/prop_spawner.cpp)
target_link_libraries(
  prop_spawner
  PUBLIC ${dependencies}
  "${PROJECT_NAME}__rosidl_typesupport_cpp"
)

add_executable(random src/random_start.cpp)
target_link_libraries(
  random
  PUBLIC ${dependencies}
  "${PROJECT_NAME}__rosidl_typesupport_cpp"
)

install(TARGETS
  ros_gz_bridge
  pusher_controller
  conveyor_controller
  prop_spawner
  random
  DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY
  config
  launch
  models
  worlds
  DESTINATION share/${PROJECT_NAME}
)

add_executable(pusher_vision src/pusher_vision.cpp)
ament_target_dependencies(pusher_vision rclcpp sensor_msgs cv_bridge OpenCV std_srvs)
install(TARGETS pusher_vision
  DESTINATION lib/${PROJECT_NAME}
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
