cmake_minimum_required(VERSION 3.15)
project(nlopt)

include(ExternalProject)

set(VERSION 2.4.2)

file(MAKE_DIRECTORY ${CMAKE_SOURCE_DIR}/include)

set(DEPENDENCIES
    ament_cmake
    rclcpp
    rclcpp_components
  )

foreach(DEP IN LISTS DEPENDENCIES)
  find_package(${DEP} REQUIRED)
endforeach()

# force the linker to link libcamera_source even when it is not needed for the wrapper lib
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-no-as-needed")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -O3")

set(NLOPT_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/nlopt_install)
set(NLOPT_INSTALL_DIR ${NLOPT_INSTALL_PREFIX}/${CMAKE_INSTALL_PREFIX})
configure_file(make_install_nlopt.sh.in ${CMAKE_BINARY_DIR}/make_install_nlopt.sh)

ExternalProject_Add(nlopt_src
  DOWNLOAD_COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/nlopt-${VERSION}.tar.gz ./
  PATCH_COMMAND tar -xzf ../nlopt-${VERSION}.tar.gz && rm -rf ../nlopt_src-build/nlopt-${VERSION} && mv nlopt-${VERSION} ../nlopt_src-build/
  CONFIGURE_COMMAND nlopt-${VERSION}/configure --with-cxx --without-matlab --with-pic --enable-shared --prefix=${CMAKE_INSTALL_PREFIX}
  BUILD_COMMAND make
  INSTALL_COMMAND ${CMAKE_BINARY_DIR}/make_install_nlopt.sh
)

include_directories(
  include
  include/nlopt_ros
  ${NLOPT_INSTALL_DIR}/include
)

set(LIBRARIES
  nlopt_wrap
  )

add_library(nlopt_wrap SHARED
  src/wrap_lib.cpp
)

target_link_libraries(nlopt_wrap PRIVATE
  ${NLOPT_INSTALL_DIR}/lib/libnlopt_cxx.so
  )

add_dependencies(nlopt_wrap
  nlopt_src
)

## --------------------------------------------------------------
## |                           install                          |
## --------------------------------------------------------------

ament_export_libraries(
  ${LIBRARIES}
)

install(TARGETS
  ${LIBRARIES}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(
  DIRECTORY include
  DESTINATION .
  )

install(
  DIRECTORY ${NLOPT_INSTALL_DIR}/lib
  DESTINATION .
  )

install(
  DIRECTORY ${NLOPT_INSTALL_DIR}/include
  DESTINATION .
  )

# install(DIRECTORY ${NLOPT_INSTALL_DIR}/
#   DESTINATION ${CMAKE_INSTALL_PREFIX}
# )

ament_export_include_directories(
  include
  )

ament_export_targets(
  export_${PROJECT_NAME} HAS_LIBRARY_TARGET
  )

ament_export_dependencies(
  ${DEPENDENCIES}
  )

ament_package()
