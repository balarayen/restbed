cmake_minimum_required( VERSION 3.1.0 FATAL_ERROR )

project( "restbed" VERSION 4.7.0 LANGUAGES CXX )
message( "Copyright 2013-2018, Corvusoft Ltd, All Rights Reserved." )

#
# Build Options
#
option( BUILD_SSL      "Build secure socket layer support."  ON )

#
# Configuration
#
set( CMAKE_CXX_STANDARD 11 )
set( INCLUDE_DIR "${PROJECT_SOURCE_DIR}/source" )
set( SOURCE_DIR  "${PROJECT_SOURCE_DIR}/source/corvusoft/${PROJECT_NAME}" )

if ( NOT DEFINED CMAKE_INSTALL_LIBDIR )
    set( CMAKE_INSTALL_LIBDIR library )
endif ( )

if ( CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT )
    set( CMAKE_INSTALL_PREFIX "${PROJECT_SOURCE_DIR}/distribution" CACHE PATH "Install path prefix" FORCE )
endif ( )

if( ${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /D_WIN32_WINNT=0x0601 /W4 /wd4068 /wd4702" )
endif ( )

if( NOT WIN32 )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Weffc++ -pedantic -Wno-unknown-pragmas" )
endif ( )

if ( UNIX AND NOT APPLE )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread" )
endif ( )

if ( APPLE )
    set( CMAKE_MACOSX_RPATH ON )
endif ( )

set( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake" )

find_package( asio REQUIRED )
find_package( catch REQUIRED )
if ( BUILD_SSL )
    find_package( openssl REQUIRED )
endif ( )

include_directories( ${INCLUDE_DIR} SYSTEM ${asio_INCLUDE} ${kashmir_INCLUDE} ${ssl_INCLUDE} )

#
# Build
#
file( GLOB_RECURSE MANIFEST "${SOURCE_DIR}/*.cpp" )

set( STATIC_LIBRARY_NAME "${PROJECT_NAME}-static" )
add_library( ${STATIC_LIBRARY_NAME} STATIC ${MANIFEST} )
set_property( TARGET ${STATIC_LIBRARY_NAME} PROPERTY CXX_STANDARD 11 )
set_property( TARGET ${STATIC_LIBRARY_NAME} PROPERTY CXX_STANDARD_REQUIRED ON )
set_target_properties( ${STATIC_LIBRARY_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME} )
if ( BUILD_SSL )
    target_link_libraries( ${STATIC_LIBRARY_NAME} LINK_PRIVATE ${ssl_LIBRARY_STATIC} ${crypto_LIBRARY_STATIC} ${CMAKE_DL_LIBS})
else ( )
    target_link_libraries( ${STATIC_LIBRARY_NAME} ${CMAKE_DL_LIBS})
endif ( )

set( SHARED_LIBRARY_NAME "${PROJECT_NAME}-shared" )
add_library( ${SHARED_LIBRARY_NAME} SHARED ${MANIFEST} )
set_property( TARGET ${SHARED_LIBRARY_NAME} PROPERTY CXX_STANDARD 11 )
set_property( TARGET ${SHARED_LIBRARY_NAME} PROPERTY CXX_STANDARD_REQUIRED ON )
set_target_properties( ${SHARED_LIBRARY_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME} )
set_target_properties( ${SHARED_LIBRARY_NAME} PROPERTIES SOVERSION ${PROJECT_VERSION_MAJOR} VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR} )
if ( BUILD_SSL )
    target_link_libraries( ${SHARED_LIBRARY_NAME} LINK_PRIVATE ${ssl_LIBRARY_SHARED} ${crypto_LIBRARY_SHARED} )
else ( )
    target_link_libraries( ${SHARED_LIBRARY_NAME} )
endif ( )

enable_testing( )
add_subdirectory( "${PROJECT_SOURCE_DIR}/test/unit" )
add_subdirectory( "${PROJECT_SOURCE_DIR}/test/feature" )
add_subdirectory( "${PROJECT_SOURCE_DIR}/test/regression" )
add_subdirectory( "${PROJECT_SOURCE_DIR}/test/integration" )

#
# Install
#
file( GLOB ARTIFACTS "${SOURCE_DIR}/*.hpp" )

install( FILES "${INCLUDE_DIR}/${PROJECT_NAME}" DESTINATION "${CMAKE_INSTALL_PREFIX}/include" )
install( FILES ${ARTIFACTS} DESTINATION "${CMAKE_INSTALL_PREFIX}/include/corvusoft/${PROJECT_NAME}" )
install( TARGETS ${STATIC_LIBRARY_NAME} LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT library )
install( TARGETS ${SHARED_LIBRARY_NAME} LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT library )
