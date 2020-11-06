cmake_minimum_required(VERSION 2.8)
project(ldso)
if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

set (CMAKE_C_CXX_COMMON_FLAGS "-Wall -Wfloat-equal -Wundef -Wno-strict-aliasing -Wno-unused-function -Wno-unused-but-set-variable -Wno-unused-macros -Wno-unknown-pragmas")
set (CMAKE_C_FLAGS "-u MAIN__ -std=gnu99 -fopenmp ${CMAKE_C_CXX_COMMON_FLAGS}")
set (CMAKE_CXX_FLAGS "-u MAIN__ -std=gnu++11 -std=c++11 -fopenmp -msse4.1 ${CMAKE_C_CXX_COMMON_FLAGS}")
set(CMAKE_CXX_FLAGS
   "${SSE_FLAGS} -O3 -g -std=c++0x -march=native"
)

find_package(cmake_modules REQUIRED)
find_package(Eigen3 REQUIRED)

set(BUILD_TYPE RelWithDebInfo)
if (MSVC)
     set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")
endif (MSVC)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)
set(CMAKE_PREFIX_PATH ${PROJECT_SOURCE_DIR}/cmake)

find_package(catkin REQUIRED COMPONENTS
	roscpp
	rospy
	rosbag
	cv_bridge

	image_transport
	std_msgs
	OpenCV
	sensor_msgs
	geometry_msgs
)

catkin_package(
	INCLUDE_DIRS include
	CATKIN_DEPENDS
	std_msgs
	cv_bridge
	image_transport
	message_runtime
	sensor_msgs
)

add_definitions(-std=c++11)

# required libs
find_package(Glog REQUIRED)
find_package(SuiteParse REQUIRED)
find_package(G2O REQUIRED)
find_package(Boost REQUIRED COMPONENTS system thread) 
find_package(Eigen3 REQUIRED)
find_package(Pangolin REQUIRED)
find_package(OpenCV 3.4.2 REQUIRED)
find_package(LibZip REQUIRED)
add_definitions(-DHAS_ZIPLIB=1)
add_definitions("-DENABLE_SSE")
set(HAS_PANGOLIN 1)
set(BOOST_THREAD_LIBRARY boost_thread)

set(CMAKE_CXX_FLAGS "-Wall -Wno-deprecated -march=native -Wno-duplicate-decl-specifier -Wno-ignored-qualifiers -Wno-reorder -Wno-missing-braces ${SSE_FLAGS} -O2 -g -std=c++0x -march=native -fno-omit-frame-pointer -pthread")

include_directories(
  		include
  		${PROJECT_SOURCE_DIR}/src
 		${PROJECT_SOURCE_DIR}/thirdparty/Sophus
        ${PROJECT_SOURCE_DIR}/thirdparty/DBoW3/
		${catkin_INCLUDE_DIRS}
        ${EIGEN3_INCLUDE_DIR}
        ${OpenCV_INCLUDE_DIR}
		${G2O_INCLUDE_DIR}
		${G2O_INCLUDE_DIRS}
        ${Glog_INCLUDE_DIRS}
        ${Pangolin_INCLUDE_DIRS}
		${CSPARSE_INCLUDE_DIR}
		${CHOLMOD_INCLUDE_DIR}
		${LIBZIP_INCLUDE_DIR_ZIP}
		${LIBZIP_INCLUDE_DIR_ZIPCONF}
)

set(HAS_OPENCV 1) #found opencv from above
include_directories( ${OpenCV_INCLUDE_DIRS} )
set(HAS_PANGOLIN 1) #found pangolin from above
include_directories( ${Pangolin_INCLUDE_DIRS} ) 

catkin_package(
  CATKIN_DEPENDS roscpp rospy cv_bridge std_msgs sensor_msgs geometry_msgs
  DEPENDS EIGEN3 Pangolin OpenCV g2o glog
  INCLUDE_DIRS include
)

SET(G2O_LIBRARIES ${G2O_STUFF_LIBRARY} ${G2O_CORE_LIBRARY} ${G2O_SOLVER_CHOLMOD} ${G2O_SOLVER_CSPARSE} ${G2O_SOLVER_DENSE} ${G2O_SOLVER_PCG} ${G2O_SOLVER_SLAM2D_LINEAR} ${G2O_SOLVER_STRUCTURE_ONLY} ${G2O_SOLVER_EIGEN} ${G2O_SOLVER_CSPARSE} ${G2O_SOLVER_CSPARSE_EXTENSION})


set( THIRD_PARTY_LIBS
        ${OpenCV_LIBS}
        ${GLOG_LIBRARY}
		${G2O_LIBRARIES}
        ${Pangolin_LIBRARIES}
        ${PROJECT_SOURCE_DIR}/thirdparty/DBoW3/DBoW3/build/src/libDBoW3${CMAKE_SHARED_LIBRARY_SUFFIX}
        ${LIBZIP_LIBRARY}
)

add_library(loopdso
        src/Feature.cc
        src/Frame.cc
        src/Point.cc
        src/Setting.cc
        src/Camera.cc
        src/Map.cc

        src/internal/PointHessian.cc
        src/internal/FrameHessian.cc
        src/internal/GlobalCalib.cc
        src/internal/FrameFramePrecalc.cc
        src/internal/Residuals.cc
        src/internal/ImmaturePoint.cc
        src/internal/PR.cc

        src/internal/OptimizationBackend/AccumulatedSCHessian.cc
        src/internal/OptimizationBackend/AccumulatedTopHessian.cc
        src/internal/OptimizationBackend/EnergyFunctional.cc

        src/frontend/CoarseTracker.cc
        src/frontend/CoarseInitializer.cc
        src/frontend/FullSystem.cc
        src/frontend/DSOViewer.cc
        src/frontend/FeatureDetector.cc
        src/frontend/FeatureMatcher.cc
        src/frontend/LoopClosing.cc
        src/frontend/PixelSelector2.cc
        src/frontend/Undistort.cc
        src/frontend/ImageRW_OpenCV.cc
)

target_link_libraries(loopdso ${THIRD_PARTY_LIBS})
add_executable(ldso src/run_ldso.cpp )
target_link_libraries(ldso loopdso ${THIRD_PARTY_LIBS})

install(FILES package.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
