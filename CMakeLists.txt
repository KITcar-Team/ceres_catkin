cmake_minimum_required(VERSION 2.8.3)
project(ceres_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple()

include_directories(${Eigen_INCLUDE_DIRS})

include(ExternalProject)

set(VERSION 1.14.0)
set(CERES_TAG facb199f3eda902360f9e1d5271372b7e54febe1) #version 1.14

# The configure step fails at catkin_package() if this directory
# doesn't exist yet. Create it at configure time!
file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)
file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include/ceres/internal/miniglog)

# Patch away the linking to the gomp OpenMP implementation, in case we use the Intel OpenMP implementation and Clang.
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
	set(GOMP_PATCH "")
else (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
	set(GOMP_PATCH "--dry-run")
endif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")

string(TOUPPER ${CMAKE_BUILD_TYPE} CMAKE_BUILD_TYPE_UPPER)
set(CXX_FLAGS ${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_${CMAKE_BUILD_TYPE_UPPER}})

ExternalProject_Add(ceres_src
  GIT_REPOSITORY https://ceres-solver.googlesource.com/ceres-solver
  GIT_TAG ${CERES_TAG}
  UPDATE_COMMAND ""
  CONFIGURE_COMMAND cmake
    -DGFLAGS=OFF
    -DCXX11=ON
    -DMINIGLOG=ON
    -DMINIGLOG_MAX_LOG_LEVEL=0
    -DBUILD_DOCUMENTATION=OFF
    -DSUITESPARSE_INCLUDE_DIR_HINTS=${CATKIN_DEVEL_PREFIX}/include/suitesparse
    -DSUITESPARSE_LIBRARY_DIR_HINTS=${CATKIN_DEVEL_PREFIX}/lib ../ceres_src
    -DBUILD_EXAMPLES=OFF
    -DBUILD_TESTING=OFF
    -DCMAKE_INSTALL_PREFIX=${CATKIN_DEVEL_PREFIX} -DBUILD_SHARED_LIBS=ON
    -DCMAKE_VERBOSE_MAKEFILE=ON
    -DCMAKE_BUILD_TYPE:STRING=Release
    -DCMAKE_CXX_FLAGS=${CXX_FLAGS}
    -DCMAKE_FIND_PACKAGE_NO_PACKAGE_REGISTRY=ON
    -DCMAKE_FIND_PACKAGE_NO_SYSTEM_PACKAGE_REGISTRY=ON
  BUILD_COMMAND make
  INSTALL_COMMAND make install
)
add_dependencies(ceres_src suitesparse)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cc)
add_dependencies(${PROJECT_NAME} ceres_src)
target_link_libraries(${PROJECT_NAME}
  ${CATKIN_DEVEL_PREFIX}/lib/libceres${CMAKE_SHARED_LIBRARY_SUFFIX})

cs_install()

cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
                       ${CATKIN_DEVEL_PREFIX}/include/ceres/internal/miniglog
          CFG_EXTRAS ceres-extras.cmake)
