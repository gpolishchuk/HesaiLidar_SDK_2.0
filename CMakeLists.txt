cmake_minimum_required(VERSION 3.8.11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 14)
PROJECT(Template)
set(VERSION_MAJOR 2)
set(VERSION_MINOR 0)
set(VERSION_TINY  1)
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/Version.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/Version.h"
)

cmake_policy(SET CMP0053 NEW)
set(CMAKE_BUILD_TYPE ON)
set(CMAKE_BUILD_TYPE Release)
add_definitions(-O3)
add_compile_options(-Wall)


include_directories("${PROJECT_BINARY_DIR}")
find_package(Boost REQUIRED COMPONENTS system filesystem thread)
add_definitions(-DQT_MESSAGELOGCONTEXT)
add_subdirectory(libhesai)

# set (Boost_INCLUDE_DIRS "/usr/lib/x86_64-linux-gnu/")

include_directories(
  libhesai
  libhesai/Lidar
  libhesai/UdpParser
  libhesai/UdpParser/include
  libhesai/UdpParser/src
  libhesai/UdpProtocol
  libhesai/Source/include
  libhesai/Container/include
  libhesai/Container/src
  libhesai/UdpParserGpu
  libhesai/UdpParserGpu/include
  libhesai/UdpParserGpu/src
  libhesai/PtcClient/include
  libhesai/Logger/include
  libhesai/include
  driver
)

link_directories(${Boost_LIBRARY_DIRS})
find_package(PCL COMPONENTS common visualization io QUIET REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_executable(sample
  test/main.cc
)

target_link_libraries(sample
  ${Boost_LIBRARIES}
  source_lib
  container_lib
  ptcClient_lib
  log_lib
  ${PCL_LIBRARIES}  
)

# add_library(libhesai
#     driver/hesai_lidar_sdk.hpp
# )
# target_link_libraries(libhesai
#   ${Boost_LIBRARIES}
#   source_lib
#   container_lib
#   ptcClient_lib
#   log_lib
  
# )

find_package(CUDA )
if(${CUDA_FOUND})
	set(CUDA_SOURCE_PROPERTY_FORMAT OBJ)
	set(CUDA_SEPARABLE_COMPILATION ON)
	include_directories(${CUDA_INCLUDE_DIRS})
	set(CUDA_PROPAGATE_HOST_FLAGS OFF)
	set(CUDA_NVCC_FLAGS -arch=sm_61;-O3;-G;-g;-std=c++14)#根据具体GPU性能更改算力参数
	link_directories($ENV{CUDA_PATH}/lib/x64)
  file(GLOB_RECURSE CURRENT_HEADERS  *.h *.hpp *.cuh)
  file(GLOB CURRENT_SOURCES  *.cpp *.cu)
  file(GLOB CURRENT_SOURCES  *.cpp *.cu)
  source_group("Include" FILES ${CURRENT_HEADERS}) 
  source_group("Source" FILES ${CURRENT_SOURCES}) 
  CUDA_ADD_EXECUTABLE(sample_gpu ${CURRENT_HEADERS} ${CURRENT_SOURCES} 
  ./test/main.cu
  ./libhesai/UdpParserGpu/src/buffer.cu)
  target_link_libraries(sample_gpu
  ${Boost_LIBRARIES}
  source_lib
  container_lib
  ptcClient_lib
  log_lib
  ${PCL_LIBRARIES} 
)
else(${CUDA_FOUND})
	MESSAGE(STATUS "cuda not found!")
endif(${CUDA_FOUND})
