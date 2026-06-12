cmake_minimum_required(VERSION 3.28)
project(nlopt)

include(ExternalProject)

set(VERSION 2.4.2)

find_package(ament_cmake REQUIRED)

file(MAKE_DIRECTORY ${CMAKE_SOURCE_DIR}/include)

set(NLOPT_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/nlopt_install)
cmake_path(SET NLOPT_INSTALL_DIR NORMALIZE "${NLOPT_INSTALL_PREFIX}/${CMAKE_INSTALL_PREFIX}")
configure_file(make_install_nlopt.sh.in ${CMAKE_BINARY_DIR}/make_install_nlopt.sh)

ExternalProject_Add(nlopt_src
  DOWNLOAD_COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/nlopt-${VERSION}.tar.gz ./
  PATCH_COMMAND tar -xzf ../nlopt-${VERSION}.tar.gz && rm -rf ../nlopt_src-build/nlopt-${VERSION} && mv nlopt-${VERSION} ../nlopt_src-build/
  CONFIGURE_COMMAND nlopt-${VERSION}/configure CXXFLAGS=-std=c++14 --with-cxx --without-matlab --with-pic --enable-shared --prefix=${CMAKE_INSTALL_PREFIX}
  BUILD_COMMAND make
  INSTALL_COMMAND ${CMAKE_BINARY_DIR}/make_install_nlopt.sh
)

add_library(nlopt_nlopt INTERFACE)
add_library(nlopt::nlopt ALIAS nlopt_nlopt)
set_target_properties(nlopt_nlopt
  PROPERTIES
    EXPORT_NAME nlopt
    OUTPUT_NAME nlopt
)

target_include_directories(nlopt_nlopt
  INTERFACE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
)

target_link_libraries(nlopt_nlopt
  INTERFACE
    $<BUILD_INTERFACE:${NLOPT_INSTALL_DIR}/lib/libnlopt_cxx.so>
    $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/lib/libnlopt_cxx.so>
)

## --------------------------------------------------------------
## |                           install                          |
## --------------------------------------------------------------

install(TARGETS
  nlopt_nlopt
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(
  DIRECTORY "${NLOPT_INSTALL_DIR}/lib/"
  DESTINATION lib
)

install(
  DIRECTORY "${NLOPT_INSTALL_DIR}/include/"
  DESTINATION "include/${PROJECT_NAME}"
)

ament_export_targets(
  export_${PROJECT_NAME} HAS_LIBRARY_TARGET
)

ament_package()
