cmake_minimum_required(VERSION 3.5.1)
project(ugv_sdk VERSION 0.1.5)

find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    message(STATUS "Found ccache")
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

## Project Options
option(BUILD_TESTS ON)
option(STATIC_CHECK OFF)
# set(CMAKE_BUILD_TYPE Release)
# set(CMAKE_BUILD_TYPE Debug)

## Check if pkg is built with ROS catkin

message(STATUS "Build package with catkin")
set(BUILD_WITHOUT_ROS OFF)
find_package(catkin REQUIRED)

## Generate symbols for IDE indexer
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(STATIC_CHECK)
    find_program(CPPCHECK cppcheck)
    if(CPPCHECK)
        message(STATUS "Found cppcheck")
        set(CMAKE_CXX_CPPCHECK cppcheck;--std=c++11;--enable=all)
    endif() 
endif()

## Additional cmake module path
set(USER_CMAKE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
list(APPEND CMAKE_MODULE_PATH "${USER_CMAKE_PATH}/modules")
list(APPEND CMAKE_PREFIX_PATH "/usr/lib/${CMAKE_SYSTEM_PROCESSOR}-linux-gnu/cmake")
list(APPEND CMAKE_PREFIX_PATH "/opt/weston_robot/lib/cmake")

## Set compiler to use c++ 11 features
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

## Chosse build type
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

find_package(catkin REQUIRED COMPONENTS wrp_io)
    include_directories(
        include
        ${catkin_INCLUDE_DIRS}
    )


# Build libraries
find_package(Threads REQUIRED)

add_library(${PROJECT_NAME} 
    # agx common
    src/mobile_base.cpp
    src/agx_msg_parser.c
    # robot support
    src/scout_base.cpp
    src/hunter_base.cpp
    src/tracer_base.cpp
    src/bunker_base.cpp
)

target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES} Threads::Threads)

target_include_directories(${PROJECT_NAME} PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:include>
    PRIVATE src)
    

# Build tests
# if(PROJECT_NAME STREQUAL CMAKE_PROJECT_NAME AND BUILD_TESTS)
#     message(STATUS "Tests will be built")
#     enable_testing()
#     include(GoogleTest)
#     add_subdirectory(tests)
# else()
#     message(STATUS "Tests will not be built")
# endif()

## Installation configuration
# BUILD_WITHOUT_ROS

catkin_package(
	LIBRARIES ${PROJECT_NAME} 
	INCLUDE_DIRS include 
	CATKIN_DEPENDS wrp_io
	#  DEPENDS system_lib
)

## Add catkin install targets
install(TARGETS ${PROJECT_NAME}
RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

install(DIRECTORY include/${PROJECT_NAME}
DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION})

# BUILD_WITHOUT_ROS
