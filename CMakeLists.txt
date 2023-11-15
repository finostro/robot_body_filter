cmake_minimum_required(VERSION 3.16.3...3.20)
project(robot_body_filter)

set(CMAKE_CXX_STANDARD 17)
set (CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(DetectOptional)

if(${ROBOT_BODY_FILTER_HAVE_CXX_OPTIONAL})
  # add_compile_definitions would be nicer, but isn't available in Stretch
  add_compile_options(-DROBOT_BODY_FILTER_USE_CXX_OPTIONAL=1)
else()
  add_compile_options(-DROBOT_BODY_FILTER_USE_CXX_OPTIONAL=0)
endif()

set(THIS_PACKAGE_DEPS dynamic_reconfigure filters geometric_shapes laser_geometry moveit_core moveit_ros_perception roscpp sensor_msgs tf2 tf2_ros urdf visualization_msgs pcl fcl Threads pluginlib)
set(MESSAGE_DEPS geometry_msgs std_msgs)

find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()



# message generation
ament_auto_generate_code()



set(UTILS_SRCS
  src/utils/bodies.cpp
  src/utils/cloud.cpp
  src/utils/shapes.cpp
  src/utils/string_utils.cpp
  src/utils/tf2_eigen.cpp
  src/utils/tf2_sensor_msgs.cpp
  src/utils/time_utils.cpp)



ament_auto_add_library(${PROJECT_NAME}_utils ${UTILS_SRCS})
target_include_directories(${PROJECT_NAME}_utils
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

ament_auto_add_library(tf2_sensor_msgs_rbf src/utils/tf2_sensor_msgs.cpp)
target_include_directories(tf2_sensor_msgs_rbf
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)
target_link_libraries(tf2_sensor_msgs_rbf PRIVATE ${PROJECT_NAME}_utils )


ament_auto_add_library(RayCastingShapeMask src/RayCastingShapeMask.cpp)
target_include_directories(RayCastingShapeMask
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)
target_link_libraries(RayCastingShapeMask ${PROJECT_NAME}_utils )

ament_auto_add_library(TFFramesWatchdog src/TFFramesWatchdog.cpp)
target_include_directories(TFFramesWatchdog
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)
target_link_libraries(TFFramesWatchdog ${PROJECT_NAME}_utils )

ament_auto_add_library(${PROJECT_NAME} src/RobotBodyFilter.cpp)
target_include_directories(TFFramesWatchdog
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

target_link_libraries(${PROJECT_NAME}
  ${PROJECT_NAME}_utils
  tf2_sensor_msgs_rbf
  TFFramesWatchdog
  RayCastingShapeMask
)

pluginlib_export_plugin_description_file(filters laser_filters.xml)
ament_auto_package(INSTALL_TO_SHARE examples)


#[[
install(DIRECTORY include/${PROJECT_NAME}/
   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

install(TARGETS ${PROJECT_NAME} TFFramesWatchdog RayCastingShapeMask ${PROJECT_NAME}_utils tf2_sensor_msgs_rbf
   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

install(FILES laser_filters.xml
    DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/)

install(FILES rviz/debug.rviz
    DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/rviz)
]]

