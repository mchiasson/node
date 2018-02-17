cmake_minimum_required(VERSION 3.5)

project(node)
include(ExternalProject)
include(ProcessorCount)

ProcessorCount(N)

set(NODE_VERSION 9.5.0)
set(NODE_LIB "lib/${CMAKE_SHARED_LIBRARY_PREFIX}node${CMAKE_SHARED_LIBRARY_SUFFIX}.61")

set(INSTALL_DIR ${CMAKE_CURRENT_SOURCE_DIR}/install)
add_custom_target(node_project ALL
    COMMAND ./configure --prefix=${INSTALL_DIR} --shared
    COMMAND make -j${N}
    COMMAND make install
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

add_library(node INTERFACE)

# Install the different headers and libraries.
install(DIRECTORY ${INSTALL_DIR}/include/node
        DESTINATION include COMPONENT Development
)
install(DIRECTORY ${INSTALL_DIR}/lib/node_modules
        DESTINATION lib COMPONENT Runtime
)
install(FILES "${INSTALL_DIR}/${NODE_LIB}"
        DESTINATION lib COMPONENT Runtime
)

# Create the CMake version file.
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/nodeConfigVersion.cmake"
  VERSION ${NODE_VERSION}
  COMPATIBILITY AnyNewerVersion
)

set(targets_export_name "nodeTargets")

install(TARGETS node
      EXPORT ${targets_export_name}
      LIBRARY DESTINATION lib COMPONENT Runtime
      ARCHIVE DESTINATION lib COMPONENT Development
      RUNTIME DESTINATION bin COMPONENT Runtime
      PUBLIC_HEADER DESTINATION include COMPONENT Development
      BUNDLE DESTINATION bin COMPONENT Runtime
)

# Create the Config file.
include(CMakePackageConfigHelpers)
set(ConfigPackageLocation lib/cmake/node)
configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/nodeConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/nodeConfig.cmake"
  INSTALL_DESTINATION ${ConfigPackageLocation}
)

# Install the generated CMake files.
install(EXPORT ${targets_export_name} NAMESPACE "node::" DESTINATION lib/cmake/node)
install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/nodeConfigVersion.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/nodeConfig.cmake"
  DESTINATION ${ConfigPackageLocation}
)
