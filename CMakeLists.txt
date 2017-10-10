cmake_minimum_required(VERSION 2.8.12)
if(NOT ITK_SOURCE_DIR)
  include(itk-module-init.cmake)
endif()
project(StatismoITK)

# When this module is loaded by an app, load statismo too.
set(StatismoITK_EXPORT_CODE_INSTALL "
set(statismo_DIR \"${statismo_DIR}\")
find_package(statismo REQUIRED)
")
set(StatismoITK_EXPORT_CODE_BUILD "
if(NOT ITK_BINARY_DIR)
  set(statismo_DIR \"${statismo_DIR}\")
  find_package(statismo REQUIRED)
endif()
")

set(StatismoITK_SYSTEM_INCLUDE_DIRS ${statismo_INCLUDE_DIRS} ${HDF5_INCLUDE_DIRS})
set(StatismoITK_SYSTEM_LIBRARY_DIRS ${Boost_LIBRARY_DIRS})
set(StatismoITK_LIBRARIES ${statismo_LIBRARIES} ${Boost_LIBRARIES} ${HDF5_LIBRARIES})

if(NOT ITK_SOURCE_DIR)
  find_package(ITK REQUIRED)
  list(APPEND CMAKE_MODULE_PATH ${ITK_CMAKE_DIR})
  include(ITKModuleExternal)
else()
  itk_module_impl()
endif()
