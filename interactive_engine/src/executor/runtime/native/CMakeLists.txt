cmake_minimum_required(VERSION 3.1)
project(gloabl_store_api C CXX)

if(POLICY CMP0025)
  cmake_policy(SET CMP0025 NEW)
endif()

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

SET(DEFAULT_BUILD_TYPE "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE
        STRING "Choose the type of build." FORCE
    )
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
        "Debug" "Release" "MinSizeRel" "RelWithDebInfo"
    )
endif()
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(BUILD_VINEYARD_PROFILING ON)
endif()

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)

# reference: https://gitlab.kitware.com/cmake/community/-/wikis/doc/cmake/RPATH-handling#always-full-rpath
include(GNUInstallDirs)
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib:${CMAKE_INSTALL_PREFIX}/lib64")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

macro(install_vineyard_target target)
    install(TARGETS ${target}
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
    )
endmacro()

SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -Wall -Wno-deprecated-declarations")

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(THREADS_PREFER_PTHREAD_FLAG TRUE)
find_package(Threads REQUIRED)

#  we need edge src/dst ids in etable.
add_definitions(-DENDPOINT_LISTS)

find_package(vineyard 0.2.9 REQUIRED)
add_library(native_store global_store_ffi.cc
                         htap_ds_impl.cc
                         graph_builder_ffi.cc
                         property_graph_stream.cc
                         graph_schema.cc
)
target_include_directories(native_store PRIVATE ${VINEYARD_INCLUDE_DIRS})
target_link_libraries(native_store ${VINEYARD_LIBRARIES})
if(BUILD_SHARED_LIBS)
    install_vineyard_target(native_store)
endif()

add_executable(vineyard_htap_reader htap_reader.c)
target_link_libraries(vineyard_htap_reader native_store ${VINEYARD_LIBRARIES} Threads::Threads)
install_vineyard_target(vineyard_htap_reader)

add_executable(vineyard_htap_mc_reader htap_mc_reader.c)
target_link_libraries(vineyard_htap_mc_reader native_store ${VINEYARD_LIBRARIES} Threads::Threads)
install_vineyard_target(vineyard_htap_mc_reader)

add_executable(vineyard_htap_loader htap_loader.cc)
target_include_directories(vineyard_htap_loader PRIVATE ${VINEYARD_INCLUDE_DIRS})
target_link_libraries(vineyard_htap_loader native_store ${VINEYARD_LIBRARIES} Threads::Threads)
install_vineyard_target(vineyard_htap_loader)

add_executable(vineyard_htap_stream_loader htap_stream_loader.cc)
target_include_directories(vineyard_htap_stream_loader PRIVATE ${VINEYARD_INCLUDE_DIRS})
target_link_libraries(vineyard_htap_stream_loader native_store ${VINEYARD_LIBRARIES} Threads::Threads)
install_vineyard_target(vineyard_htap_stream_loader)

add_executable(vineyard_htap_stream_generator htap_stream_generator.cc)
target_include_directories(vineyard_htap_stream_generator PRIVATE ${VINEYARD_INCLUDE_DIRS})
target_link_libraries(vineyard_htap_stream_generator native_store ${VINEYARD_LIBRARIES} Threads::Threads)
install_vineyard_target(vineyard_htap_stream_generator)
