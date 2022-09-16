cmake_minimum_required(VERSION 3.2)
project(GC_LOAM)  # this is a slam modified from lio-sam for 6-dof imu

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++11")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g -pthread")

find_package(catkin REQUIRED COMPONENTS
  tf
  roscpp
  rospy
  cv_bridge
  # pcl library
  pcl_conversions
  # msgs
  std_msgs
  sensor_msgs
  geometry_msgs
  nav_msgs
  message_generation
  visualization_msgs
)

find_package(OpenMP REQUIRED)
find_package(PCL REQUIRED QUIET)
find_package(OpenCV REQUIRED QUIET)
# add third part conf
set(3RDPARTY_DIR ${PROJECT_SOURCE_DIR}/../../../workspace/3rdparty)

set(GFLAGS_INCLUDE_DIRS ${3RDPARTY_DIR}/gflags2.2.2/include)
set(GFLAGS_LIBRARY_DIRS ${3RDPARTY_DIR}/gflags2.2.2/lib)
file(GLOB GFLAGS_LIBRARIES ${GFLAGS_LIBRARY_DIRS}/*.a ${GFLAGS_LIBRARY_DIRS}/*.so*)

set(GLOG_INCLUDE_DIRS ${3RDPARTY_DIR}/glog040/include)
set(GLOG_LIBRARY_DIRS ${3RDPARTY_DIR}/glog040/lib)
file(GLOB GLOG_LIBRARIES ${GLOG_LIBRARY_DIRS}/*.a ${GLOG_LIBRARY_DIRS}/*.so*)


# find_package(Ceres REQUIRED PATHS /home/cc/workspace/3rdparty/ceres_1.14)
set(CERES_INCLUDE_DIRS ${3RDPARTY_DIR}/ceres_1.14/include)
set(CERES_LIBRARY_DIRS ${3RDPARTY_DIR}/ceres_1.14/lib)
file(GLOB CERES_LIBRARIES ${CERES_LIBRARY_DIRS}/*.a ${CERES_LIBRARY_DIRS}/*.so*)

message(STATUS "Using catkin version ${catkin_VERSION}")
message(STATUS "Using Ceres version ${CERES_VERSION}")

# find_package(GTSAM REQUIRED QUIET)
#algo inc & lib
set(GTSAM_INCLUDE_DIR /home/cc/workspace/3rdparty/gtsam4.2a0/include)
set(GTSAM_LIBRARY_DIRS /home/cc/workspace/3rdparty/gtsam4.2a0/lib)
file(GLOB GTSAM_LIBRARIES ${GTSAM_LIBRARY_DIRS}/*.a ${GTSAM_LIBRARY_DIRS}/*.so*)

# find_package(Ceres REQUIRED)
# find_package(SuiteSparse REQUIRED)

add_message_files(
  DIRECTORY msg
  FILES
  cloud_info.msg
)

add_service_files(
  DIRECTORY srv
  FILES
  save_map.srv
)

generate_messages(
  DEPENDENCIES
  geometry_msgs
  std_msgs
  nav_msgs
  sensor_msgs
)

catkin_package(
  INCLUDE_DIRS include
  DEPENDS PCL GTSAM

  CATKIN_DEPENDS
  std_msgs
  nav_msgs
  geometry_msgs
  sensor_msgs
  message_runtime
  message_generation
  visualization_msgs
)

# include directories
include_directories(
	include
	${catkin_INCLUDE_DIRS}
	${PCL_INCLUDE_DIRS}
  ${GLOG_INCLUDE_DIRS}
  ${GFLAGS_INCLUDE_DIRS}
  ${CERES_INCLUDE_DIRS}
  ${SUITESPARSE_INCLUDE_DIRS}
  
  ${OpenCV_INCLUDE_DIRS}
	${GTSAM_INCLUDE_DIR}
)

# link directories
link_directories(
	include
	${PCL_LIBRARY_DIRS}
  ${OpenCV_LIBRARY_DIRS}
  ${GFLAGS_LIBRARY_DIRS}
  ${GLOG_LIBRARY_DIRS}
  ${CERES_LIBRARY_DIRS}
  ${GTSAM_LIBRARY_DIRS}
)

###########
## Build ##
###########

# Range Image Projection
add_executable(${PROJECT_NAME}_imageProjection src/imageProjection.cpp)
add_dependencies(${PROJECT_NAME}_imageProjection ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_link_libraries(${PROJECT_NAME}_imageProjection ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})

# Feature Association
add_executable(${PROJECT_NAME}_featureExtraction src/featureExtraction.cpp)
add_dependencies(${PROJECT_NAME}_featureExtraction ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_link_libraries(${PROJECT_NAME}_featureExtraction ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})

# Mapping Optimization
add_executable(${PROJECT_NAME}_mapOptmization src/mapOptmization.cpp)
add_dependencies(${PROJECT_NAME}_mapOptmization ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_compile_options(${PROJECT_NAME}_mapOptmization PRIVATE ${OpenMP_CXX_FLAGS})
target_link_libraries(${PROJECT_NAME}_mapOptmization ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES} ${OpenMP_CXX_FLAGS} gtsam libtbb.so)

# IMU Preintegration
add_executable(${PROJECT_NAME}_imuPreintegration src/imuPreintegration.cpp)
target_link_libraries(${PROJECT_NAME}_imuPreintegration ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES} gtsam libtbb.so)

# 
# add_executable(${PROJECT_NAME}_featureExtract src/lio/featureExtract.cpp)
# target_link_libraries(${PROJECT_NAME}_featureExtract ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})

# add_executable(${PROJECT_NAME}_poseEstimate src/lio/PoseEstimation.cpp src/lio/Estimator.cpp src/lio/IMUIntegrator.cpp)
# target_link_libraries(${PROJECT_NAME}_poseEstimate 
#                       ${catkin_LIBRARIES}  
#                       ${PCL_LIBRARIES} 
#                       ${OpenCV_LIBRARIES} 
#                       ${GLOG_LIBRARIES} 
#                       ${GFLAGS_LIBRARIES} 
#                       ${SUITESPARSE_LIBRARIES}
#                       ${CHOLMOD_LIBRARY}
#                       ${CERES_LIBRARIES} libgomp.so libblas.so liblapack.so)