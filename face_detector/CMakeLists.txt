cmake_minimum_required(VERSION 2.8.3)
project(face_detector)
# Build options
option(BUILD_EXAMPLES  "Set to ON to build examples"  ON)
option(USE_OPENMP      "Set to ON to build use openmp"  ON)
option(USE_SSE         "Set to ON to build use SSE"  ON)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2")

# Use C++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
message(STATUS "C++11 support has been enabled by default.")
# Use SSE
if (USE_SSE)
    add_definitions(-DUSE_SSE)
    message(STATUS "Use SSE")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse4.1")
endif()

# Use OpenMP
if (USE_OPENMP)
    find_package(OpenMP QUIET)
    if (OPENMP_FOUND)
        message(STATUS "Use OpenMP")
        add_definitions(-DUSE_OPENMP)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
    endif()
endif()

find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  image_transport
  roscpp
  rospy
  sensor_msgs
  std_msgs
  OpenCV
)


catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES imgtransport
#  CATKIN_DEPENDS cv_bridge image_transport roscpp sensor_msgs std_msgs
#  DEPENDS system_lib
)
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)
set(src_files 
    src/util/nms.cpp
    src/util/image_pyramid.cpp
    src/io/lab_boost_model_reader.cpp
    src/io/surf_mlp_model_reader.cpp
    src/feat/lab_feature_map.cpp
    src/feat/surf_feature_map.cpp
    src/classifier/lab_boosted_classifier.cpp
    src/classifier/mlp.cpp
    src/classifier/surf_mlp.cpp
    src/face_detection.cpp
    src/fust.cpp
    )
add_library(seeta_facedet_lib SHARED ${src_files})
set(facedet_required_libs seeta_facedet_lib)
list(APPEND facedet_required_libs ${OpenCV_LIBS})


add_executable(facedetector src/test/facedetection_test.cpp)
target_link_libraries(facedetector ${facedet_required_libs}
${catkin_LIBRARIES})

