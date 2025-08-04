cmake_minimum_required(VERSION 3.0)
project(unitree_z1_combined LANGUAGES CXX)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -std=c++14 -pthread")

# ==============================
# Find Dependencies
# ==============================
find_package(Boost REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(pybind11 QUIET)
find_package(catkin QUIET)

# ==============================
# Include & Link Directories
# ==============================
include_directories(
    ${Boost_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIRS}
    include
)

# Add Eigen manually if needed
include_directories("/usr/include/eigen3")

link_directories(lib)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

# ==============================
# SDK Example Programs
# ==============================
set(SDK_EXAMPLES
    examples/highcmd_basic.cpp
    examples/highcmd_development.cpp
    examples/lowcmd_development.cpp
    examples/lowcmd_multirobots.cpp
)

foreach(EXAMPLE_FILE IN LISTS SDK_EXAMPLES)
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_FILE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_FILE})
    target_link_libraries(${EXAMPLE_NAME} Z1_SDK_${CMAKE_SYSTEM_PROCESSOR})
endforeach()

# ==============================
# Python Interface (Pybind11)
# ==============================
if(pybind11_FOUND)
    pybind11_add_module(unitree_arm_interface examples_py/arm_python_interface.cpp)
    target_link_libraries(unitree_arm_interface PRIVATE Z1_SDK_${CMAKE_SYSTEM_PROCESSOR})
endif()

# ==============================
# Controller Binary (main.cpp)
# ==============================
add_executable(z1_ctrl z1_controller/main.cpp)
target_link_libraries(z1_ctrl libZ1_${CMAKE_HOST_SYSTEM_PROCESSOR}.so)

# ==============================
# Add Simulation if ROS/Catkin found
# ==============================
if(catkin_FOUND)
    add_subdirectory(z1_controller/sim)
endif()
