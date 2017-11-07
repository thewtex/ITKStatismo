cmake_minimum_required(VERSION 3.3.0)
project(StatismoITK)

if(NOT ITK_SOURCE_DIR)
  find_package(ITK REQUIRED)
endif()
if(DEFINED ITK_DIR)
  set(_itk_dir ${ITK_DIR})
else()
  # ITK in source build as a Remote Module
  set(_itk_dir ${ITK_BINARY_DIR})
endif()

if(DEFINED statismo_DIR)
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
else()
  include(ExternalProject)

  if(NOT WIN32)
    set(_lib_prefix lib)
    if(BUILD_SHARED_LIBS)
      set(_lib_suffix "${CMAKE_SHARED_LIBRARY_SUFFIX}")
      set(_lib_prefix "${CMAKE_SHARED_LIBRARY_PREFIX}")
    else()
      set(_lib_suffix "${CMAKE_STATIC_LIBRARY_SUFFIX}")
      set(_lib_prefix "${CMAKE_STATIC_LIBRARY_PREFIX}")
    endif()
  else()
    set(_lib_prefix "")
    if(BUILD_SHARED_LIBS)
      set(_lib_suffix "${CMAKE_IMPORT_LIBRARY_SUFFIX}")
      set(_lib_prefix "${CMAKE_IMPORT_LIBRARY_PREFIX}")
    else()
      set(_lib_suffix "${CMAKE_STATIC_LIBRARY_SUFFIX}")
      set(_lib_prefix "${CMAKE_IMPORT_LIBRARY_PREFIX}")
    endif()
  endif()

  set(statismo_DIR "${CMAKE_CURRENT_BINARY_DIR}/statismo-build/Statismo-build")
  set(statismo_LIBRARIES statismo_core)
  set(statismo_INCLUDE_DIRS "${CMAKE_CURRENT_BINARY_DIR}/statismo/modules/core/include")
  foreach(_lib ${statismo_LIBRARIES})
    add_library(${_lib} STATIC IMPORTED GLOBAL)
  endforeach()
  set(_lib_dir "${CMAKE_CURRENT_BINARY_DIR}/statismo-build/Statismo-build/modules/core/src")

  if(CMAKE_CONFIGURATION_TYPES)
    set(StatismoITK_EXPORT_CODE_BUILD "
foreach(_lib ${statismo_LIBRARIES})
  if(NOT TARGET \${_lib})
    add_library(\${_lib} STATIC IMPORTED)
    foreach(c ${CMAKE_CONFIGURATION_TYPES})
      string(TOUPPER \"\${c}\" C)
      set_property(TARGET \${_lib} PROPERTY IMPORTED_LOCATION_\${C}
        \"${_lib_dir}/\${c}/${_lib_prefix}\${_lib}${_lib_suffix}\")
    endforeach()
  endif()
endforeach()
")
  else()
    set(StatismoITK_EXPORT_CODE_BUILD "
foreach(_lib ${statismo_LIBRARIES})
  if(NOT TARGET \${_lib})
    add_library(\${_lib} STATIC IMPORTED)
    set_property(TARGET \${_lib} PROPERTY IMPORTED_LOCATION
      \"${_lib_dir}/${_lib_prefix}\${_lib}${_lib_suffix}\")
  endif()
endforeach()
")
  endif()
  set(StatismoITK_EXPORT_CODE_INSTALL "
foreach(_lib ${statismo_LIBRARIES})
  if(NOT TARGET \${_lib})
    add_library(\${_lib} STATIC IMPORTED)
    set_property(TARGET \${_lib} PROPERTY IMPORTED_LOCATION
      \"\${ITK_INSTALL_PREFIX}/lib/${_lib_prefix}\${_lib}${_lib_suffix}\")
  endif()
endforeach()
")

  # Attach actual filenames to the
  # imported libraries from the ExternalProject
  set(build_byproducts BUILD_BYPRODUCTS)
  foreach(_lib ${statismo_LIBRARIES})
    # tell the imported library where it's file lives
    if(CMAKE_CONFIGURATION_TYPES)
      foreach(c ${CMAKE_CONFIGURATION_TYPES})
        string(TOUPPER "${c}" C)
        set_property(TARGET ${_lib} PROPERTY IMPORTED_LOCATION_${C}
          ${_lib_dir}/${c}/${_lib_prefix}${_lib}${_lib_suffix})
        list(APPEND build_byproducts "${_lib_dir}/${c}/${_lib_prefix}${_lib}${_lib_suffix}")
      endforeach()
    else()
      set_property(TARGET ${_lib} PROPERTY IMPORTED_LOCATION
        ${_lib_dir}/${_lib_prefix}${_lib}${_lib_suffix})
      list(APPEND build_byproducts "${_lib_dir}/${_lib_prefix}${_lib}${_lib_suffix}")
    endif()
  endforeach()

  ExternalProject_Add(StatismoSuperbuild
    GIT_REPOSITORY https://github.com/thewtex/statismo.git
    GIT_TAG itk-vtk-superbuild-optional
    SOURCE_DIR statismo
    BINARY_DIR statismo-build
    SOURCE_SUBDIR superbuild
    INSTALL_COMMAND ""
    CMAKE_CACHE_ARGS
      -DCMAKE_CXX_COMPILER:FILEPATH=${CMAKE_CXX_COMPILER}
      -DCMAKE_CXX_FLAGS:STRING=${CMAKE_CXX_FLAGS}
      -DCMAKE_C_COMPILER:FILEPATH=${CMAKE_C_COMPILER}
      -DCMAKE_C_FLAGS:STRING=${CMAKE_C_FLAGS}
      -DITK_DIR:PATH=${_itk_dir}
      -DHDF5_LIBRARIES:STRING=${ITKHDF5_LIBRARIES}
      -DHDF5_INCLUDE_DIRS:STRING=${ITKHDF5_INCLUDE_DIRS}
      -DUSE_SYSTEM_HDF5:BOOL=ON
    ${build_byproducts}
    )
  foreach(_lib ${statismo_LIBRARIES})
    # Make imported library target depend on external project target
    add_dependencies(${_lib} StatismoSuperbuild)
  endforeach()

  # Run Statismo's CMake install script on install
  #install(SCRIPT
    #"${CMAKE_CURRENT_BINARY_DIR}/statismo-build/cmake_install.cmake")

  add_definitions( -DBOOST_THREAD_VERSION=3 )
  add_definitions( -DBOOST_FILESYSTEM_VERSION=3 )
  if(MSVC11) #i.e. Visual Studio 2012
    # Fix for VS2012 that has _VARIADIC_MAX set to 5. Don't set too high because it increases compiler memory usage / compile-time.
    add_definitions( -D_VARIADIC_MAX=10 )
    # Fix for another VS2012 problem: not all TR1 options are automatically detected, therefore we force them here.
    add_definitions( -D BOOST_HAS_TR1 )
    add_definitions( -D BOOST_NO_0X_HDR_INITIALIZER_LIST )
  endif()
endif()

if(NOT ITK_SOURCE_DIR)
  include(itk-module-init.cmake)
endif()

set(StatismoITK_SYSTEM_INCLUDE_DIRS ${statismo_INCLUDE_DIRS})
set(StatismoITK_SYSTEM_LIBRARY_DIRS ${Boost_LIBRARY_DIRS})
set(StatismoITK_LIBRARIES ${statismo_LIBRARIES} ${Boost_LIBRARIES} ${HDF5_LIBRARIES})

if(NOT ITK_SOURCE_DIR)
  find_package(ITK REQUIRED)
  list(APPEND CMAKE_MODULE_PATH ${ITK_CMAKE_DIR})
  include(ITKModuleExternal)
else()
  itk_module_impl()
endif()
