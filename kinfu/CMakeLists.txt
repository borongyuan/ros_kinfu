cmake_minimum_required(VERSION 2.8.3)
project(kinfu)

if (CMAKE_COMPILER_IS_GNUCC AND (NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 6.0))
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++98")
  message(STATUS "kinfu: forcing c++98 for CUDA compatibility.")
endif()

find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  cmake_modules
  image_transport
  kinfu_msgs
  message_filters
  roscpp
  sensor_msgs
  std_msgs
  tf
  pcl_msgs
  pcl_conversions
)

find_package(
  PCL 1.8 REQUIRED HINTS "/usr/local/taraxl-pcl"
  )

find_package(
  Eigen REQUIRED
  )

set(KINFUTRIGGER_FILE "cmake/KinfuTrigger.cmake")

if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${KINFUTRIGGER_FILE}")
  set(KINFU_CONFIGURED true)
  message(STATUS "kinfu: file ${KINFUTRIGGER_FILE} found, kinfu will be built.")
  add_subdirectory(pcl_kinfu_large_scale)
  include("${CMAKE_CURRENT_SOURCE_DIR}/${KINFUTRIGGER_FILE}")
else()
  set(KINFU_CONFIGURED false)
  message(WARNING "kinfu: file ${KINFUTRIGGER_FILE} not found, kinfu won't be built.")
endif()

################################################
## Declare ROS messages, services and actions ##
################################################

###################################
## catkin specific configuration ##
###################################
catkin_package(
  CATKIN_DEPENDS geometry_msgs image_transport kinfu_msgs message_filters roscpp sensor_msgs std_msgs tf
)

###########
## Build ##
###########

include_directories(
  include
  ${KinfuLargeScale_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${Eigen_INCLUDE_DIRS}
)

link_directories(${PCL_LIBRARY_DIRS})

if (${KINFU_CONFIGURED})
  add_executable(kinfu
    src/commandsubscriber.cpp
    src/kinfuLS.cpp
    src/weightcubelistener.cpp
    src/worlddownloadmanager.cpp
    src/worlddownloadutils.cpp
    src/incompletepointslistener.cpp

    src/kinfu_output_request_manager.cpp
    src/kinfu_output_action_manager.cpp
    )

  add_dependencies(kinfu kinfu_msgs_generate_messages_cpp)

  target_link_libraries(kinfu
    boost_system boost_signals
    ${CUDA_LIBRARIES}
    ${PCL_LIBRARIES}
    ${catkin_LIBRARIES}
    ${KinfuLargeScale_LIBRARIES}
    ${Eigen_LIBRARIES}
  )
endif()

#############
## Install ##
#############

if (${KINFU_CONFIGURED})
  install(TARGETS kinfu
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  )

  install(DIRECTORY launch/
    DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
  )
endif()

#############
## Testing ##
#############
