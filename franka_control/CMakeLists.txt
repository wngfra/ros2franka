cmake_minimum_required(VERSION 3.5)
project(franka_control)

list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_CURRENT_LIST_DIR}/cmake)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -O3)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
# uncomment the following section in order to fill in
# further dependencies manually.
find_package(franka_interfaces REQUIRED)
find_package(tactile_interfaces REQUIRED)
find_package(rclcpp REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Franka 0.8.0 REQUIRED)

# set(THREADS_PREFER_PTHREAD_FLAG ON)
# find_package(Threads REQUIRED)

include_directories(
  include
)

# add franka libraries
add_library(common STATIC
  src/common.cpp
)

add_library(motion_controller STATIC
  src/MotionController.cpp
)

target_link_libraries(common PUBLIC Eigen3::Eigen3 Franka::Franka)

target_link_libraries(motion_controller PUBLIC common Eigen3::Eigen3 Franka::Franka)

# add ROS2 libraries 
add_library(motion_control_server STATIC
  src/MotionControlServer.cpp
)

target_link_libraries(motion_control_server
  common
  motion_controller
  Eigen3::Eigen3
  Franka::Franka
)

ament_target_dependencies(motion_control_server
  "franka_interfaces"
  "rclcpp"
)

# Build apps
set(APPS
  node_launcher
)

foreach(target ${APPS})
  add_executable(${target} src/${target}.cpp)
  target_link_libraries(${target}
    motion_control_server)
  ament_target_dependencies(${target}
    "rclcpp")
  install(TARGETS ${target}
    DESTINATION lib/${PROJECT_NAME})
endforeach()

# Install launch files
install(DIRECTORY
  launch
  DESTINATION share/${PROJECT_NAME}/
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
