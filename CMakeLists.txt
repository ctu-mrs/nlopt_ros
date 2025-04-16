cmake_minimum_required(VERSION 3.15)
project(nlopt)

include(ExternalProject)

set(VERSION 2.4.2)

file(MAKE_DIRECTORY ${CMAKE_SOURCE_DIR}/include)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_components REQUIRED)


set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -O3")

ExternalProject_Add(nlopt_src
  DOWNLOAD_COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/nlopt-${VERSION}.tar.gz ./
  PATCH_COMMAND tar -xzf ../nlopt-${VERSION}.tar.gz && rm -rf ../nlopt_src-build/nlopt-${VERSION} && mv nlopt-${VERSION} ../nlopt_src-build/
  CONFIGURE_COMMAND nlopt-${VERSION}/configure --with-cxx --without-matlab --with-pic --prefix=${CMAKE_INSTALL_PREFIX}/tmp
  BUILD_COMMAND make
  INSTALL_COMMAND make install
)

set(LIBRARIES
  nlopt_wrap
  )

add_library(nlopt_wrap
  src/wrap_lib.cc
)

if(APPLE)
  target_link_libraries(nlopt_wrap -Wl,-all_load ${CMAKE_INSTALL_PREFIX}/tmp/lib/libnlopt_cxx.a -Wl,-noall_load)
else()
  target_link_libraries(nlopt_wrap -Wl,--whole-archive ${CMAKE_INSTALL_PREFIX}/tmp/lib/libnlopt_cxx.a -Wl,--no-whole-archive)
endif()

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
  DIRECTORY ${CMAKE_INSTALL_PREFIX}/tmp/lib
  DESTINATION lib
  )

install(
  DIRECTORY ${CMAKE_INSTALL_PREFIX}/tmp/include
  DESTINATION include
  )

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
