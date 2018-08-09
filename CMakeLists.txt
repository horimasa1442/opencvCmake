cmake_minimum_required(VERSION 3.1.0)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/../cmake ${CMAKE_SOURCE_DIR}/cmake)

project(testMain VERSION 0.4.0 LANGUAGES CXX)
add_executable(testMain main.cpp)

# Options
option(BUILD_SHARED_LIBS "Build libraries as SHARED" OFF)
set(DOWNLOAD_DIR ${CMAKE_BINARY_DIR}/download CACHE PATH "Third-party download directory")

# For external projects
set(INSTALL_DIR ${CMAKE_BINARY_DIR}/install)
set(DEFAULT_ARGS
  -DCMAKE_PREFIX_PATH=${INSTALL_DIR};${CMAKE_PREFIX_PATH}
  -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
  -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS}
)
if(CMAKE_BUILD_TYPE)
  list(APPEND DEFAULT_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
endif()
if(MSVC)
  list(APPEND DEFAULT_ARGS "-DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS} /wd4819 /wd4267 /wd4290 /wd4996")
endif()

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX ${INSTALL_DIR} CACHE PATH "Onlinesfm install prefix" FORCE)
endif()

include(ExternalProject)

# Eigen
ExternalProject_Add(eigen
  DOWNLOAD_DIR ${DOWNLOAD_DIR}
  URL http://bitbucket.org/eigen/eigen/get/3.3.5.tar.gz
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
ExternalProject_Get_Property(eigen source_dir)
set(EIGEN_INCLUDE_DIR ${source_dir})


# OpenCV
set(OCV_ARGS
  -DBUILD_DOCS=OFF
  -DBUILD_PERF_TESTS=OFF
  -DBUILD_TESTS=OFF
  -DBUILD_opencv_apps=OFF
  -DBUILD_opencv_java=OFF
  -DBUILD_opencv_python2=OFF
  -DBUILD_opencv_shape=OFF
  -DBUILD_opencv_stitching=OFF
  -DBUILD_opencv_superres=OFF
  -DBUILD_opencv_ts=OFF
  -DBUILD_opencv_videostab=OFF
  -DBUILD_WITH_STATIC_CRT=OFF
  -DWITH_CUDA=OFF
  -DWITH_EIGEN=ON
  -DWITH_TBB=OFF
  -DWITH_MATLAB=OFF
  -DEIGEN_INCLUDE_PATH=${EIGEN_INCLUDE_DIR}
)
ExternalProject_Add(opencv
  DEPENDS eigen
  SOURCE_DIR ${DOWNLOAD_DIR}/opencv
  GIT_REPOSITORY https://github.com/Itseez/opencv.git
  CMAKE_ARGS ${DEFAULT_ARGS} ${OCV_ARGS}
)

#message(${OpenCV_INCLUDE_DIRS})
#message(${OpenCV_LIBS})

add_dependencies(testMain eigen)
add_dependencies(testMain opencv)

ExternalProject_Get_Property(opencv source_dir)
set(OpenCV_INCLUDE_DIRS ${source_dir})

ExternalProject_Get_Property(opencv binary_dir)
set(OpenCV_LIBS ${binary_dir})

message(${OpenCV_INCLUDE_DIRS})
message(${OpenCV_LIBS})

target_include_directories(testMain PRIVATE 
  ${OpenCV_INCLUDE_DIRS}
  ${EIGEN_INCLUDE_DIR}
  )
  
target_link_libraries(testMain PRIVATE
  ${OpenCV_LIBS}
)