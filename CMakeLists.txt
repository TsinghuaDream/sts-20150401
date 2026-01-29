
cmake_minimum_required(VERSION 3.12 FATAL_ERROR)

# Enable find_package to use <PackageName>_ROOT variables
cmake_policy(SET CMP0074 NEW)

project(alibabacloud_sts20150401 VERSION "1.0.0")

# Build type options
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(BUILD_TESTS "Build tests" OFF)

# <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< General set up >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> #
if (CMAKE_CXX_STANDARD)
   SET(CMAKE_CXX_STANDARD ${CMAKE_CXX_STANDARD})
   message(STATUS "Using C++ standard: ${CMAKE_CXX_STANDARD}")
else()
   SET(CMAKE_CXX_STANDARD 11)
   message(STATUS "Using C++ standard: 11")
endif()

# Allow user to override CMAKE_INSTALL_PREFIX
if(NOT CMAKE_INSTALL_PREFIX OR CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "${PROJECT_SOURCE_DIR}/${PROJECT_NAME}_build" CACHE PATH "Install path" FORCE)
endif()

SET(CMAKE_FIND_LIBRARY_SUFFIXES "${CMAKE_FIND_LIBRARY_SUFFIXES}" "_d.dylib")

# Prefer static libraries when BUILD_SHARED_LIBS=OFF
if(NOT BUILD_SHARED_LIBS)
  set(OPENSSL_USE_STATIC_LIBS ON)
  list(INSERT CMAKE_FIND_LIBRARY_SUFFIXES 0 ".a")
endif()

message("C++ compiler flags: ${CMAKE_CXX_FLAGS}")

if (CMAKE_CXX_COMPILER_LOADED)
    message(STATUS "The C++ compiler ID is: ${CMAKE_CXX_COMPILER_ID}")
    message(STATUS "The C++ compiler version is: ${CMAKE_CXX_COMPILER_VERSION}")
endif ()

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif ()
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

# <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< Target set up >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> #=
include_directories(${CMAKE_SOURCE_DIR}/include)

set(SOURCE_FILES
        src/Client.cpp)

add_library(${PROJECT_NAME} ${SOURCE_FILES})
        
target_include_directories(${PROJECT_NAME}
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

set_target_properties(${PROJECT_NAME}
        PROPERTIES
        POSITION_INDEPENDENT_CODE 1
        SOVERSION ${PROJECT_VERSION_MAJOR}
        OUTPUT_NAME "${PROJECT_NAME}"
        DEBUG_POSTFIX "_d"
        PUBLIC_HEADER "${headers}"
        MACOSX_RPATH ON
        WINDOWS_EXPORT_ALL_SYMBOLS ON)

# <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< External set up >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> #
add_subdirectory(external)

# Find dependencies with proper target support
find_package(OpenSSL REQUIRED)
if(NOT TARGET OpenSSL::SSL)
  message(FATAL_ERROR "OpenSSL::SSL target not found")
endif()

# Ensure downstream dependency targets see OpenSSL headers
if(OPENSSL_INCLUDE_DIR)
  if(TARGET darabonba_core)
    target_include_directories(darabonba_core PUBLIC ${OPENSSL_INCLUDE_DIR})
  endif()
  if(TARGET alibabacloud_credential)
    target_include_directories(alibabacloud_credential PUBLIC ${OPENSSL_INCLUDE_DIR})
  endif()
endif()

find_package(CURL)
if(TARGET CURL::libcurl)
  set(CURL_TARGET CURL::libcurl)
elseif(CURL_LIBRARIES)
  set(CURL_TARGET ${CURL_LIBRARIES})
else()
  find_library(CURL_LIB curl REQUIRED)
  set(CURL_TARGET ${CURL_LIB})
endif()

if(CMAKE_HOST_WIN32)
  ExternalProject_Get_Property(curl INSTALL_DIR)
    set(curl_install_dir ${INSTALL_DIR})
    add_dependencies(${PROJECT_NAME}
            curl)
    target_include_directories(${PROJECT_NAME}
            PRIVATE	${curl_install_dir}/include)
    target_link_libraries(${PROJECT_NAME}
            OpenSSL::SSL OpenSSL::Crypto
            Crypt32
            Rpcrt4
            Ws2_32
            debug ${curl_install_dir}/lib/libcurl-d.lib
            optimized ${curl_install_dir}/lib/libcurl.lib
            darabonba_core
            alibabacloud_open_api_v2
            alibabacloud_credential)
    target_compile_definitions(${PROJECT_NAME}
            PRIVATE CURL_STATICLIB
            _CRT_SECURE_NO_WARNINGS)
elseif(APPLE)
  find_library(CFLIB CoreFoundation REQUIRED)
  target_link_libraries(${PROJECT_NAME}
      OpenSSL::SSL OpenSSL::Crypto
      ${CFLIB}
      ${CURL_TARGET}
      darabonba_core
      alibabacloud_open_api_v2
      alibabacloud_credential)
  target_compile_definitions(${PROJECT_NAME} PRIVATE GUID_CFUUID)
else()
  # Linux
  target_link_libraries(${PROJECT_NAME}
      OpenSSL::SSL OpenSSL::Crypto
      ${CURL_TARGET}
      darabonba_core
      alibabacloud_open_api_v2
      alibabacloud_credential)
endif()


# <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< Install set up >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> #
message(STATUS "${PROJECT_NAME} : Project will be installed to ${CMAKE_INSTALL_PREFIX}")

include(GNUInstallDirs)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})

set(INSTALL_LIBDIR ${CMAKE_INSTALL_LIBDIR} CACHE PATH "Installation directory for libraries")
set(INSTALL_BINDIR ${CMAKE_INSTALL_BINDIR} CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDEDIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE PATH "Installation directory for header files")
if(WIN32 AND NOT CYGWIN)
    set(DEF_INSTALL_CMAKEDIR CMake)
else()
    set(DEF_INSTALL_CMAKEDIR share/cmake/${PROJECT_NAME})
endif()
set(INSTALL_CMAKEDIR ${DEF_INSTALL_CMAKEDIR} CACHE PATH "Installation directory for CMake files")

foreach(p LIB BIN INCLUDE CMAKE)
    file(TO_NATIVE_PATH ${CMAKE_INSTALL_PREFIX}/${INSTALL_${p}DIR} _path )
    message(STATUS "Installing ${p} components to ${_path}")
    unset(_path)
endforeach()

# Prepare RPATH
file(RELATIVE_PATH _rel ${CMAKE_INSTALL_PREFIX}/${INSTALL_BINDIR} ${CMAKE_INSTALL_PREFIX})
if(APPLE)
    set(_rpath "@loader_path/${_rel}")
else()
    set(_rpath "\$ORIGIN/${_rel}")
endif()
file(TO_NATIVE_PATH "${_rpath}/${INSTALL_LIBDIR}" install_RPATH)

set_target_properties(${PROJECT_NAME}
        PROPERTIES
        MACOSX_RPATH ON
        SKIP_BUILD_RPATH OFF
        BUILD_WITH_INSTALL_RPATH OFF
        INSTALL_RPATH "${install_RPATH}"
        INSTALL_RPATH_USE_LINK_PATH ON)

install(
        TARGETS
        ${PROJECT_NAME}
        EXPORT
        alibabacloud_sts20150401Targets
        ARCHIVE
        DESTINATION ${INSTALL_LIBDIR}
        COMPONENT lib
        RUNTIME
        DESTINATION ${INSTALL_BINDIR}
        COMPONENT bin
        LIBRARY
        DESTINATION ${INSTALL_LIBDIR}
        COMPONENT lib
)

# Install header files
install(
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${INSTALL_INCLUDEDIR}
        COMPONENT dev
        FILES_MATCHING PATTERN "*.hpp"
)

# Install dependency headers if built via FetchContent
# FetchContent sets <lowercaseName>_SOURCE_DIR variables
if(DEFINED _darabonba_core_SOURCE_DIR)
  message(STATUS "Installing darabonba_core headers from: ${_darabonba_core_SOURCE_DIR}")
  install(
    DIRECTORY ${_darabonba_core_SOURCE_DIR}/include/
    DESTINATION ${INSTALL_INCLUDEDIR}
    COMPONENT dev
    FILES_MATCHING PATTERN "*.hpp" PATTERN "*.h"
  )
endif()

if(DEFINED _alibabacloud_credential_SOURCE_DIR)
  message(STATUS "Installing credential headers from: ${_alibabacloud_credential_SOURCE_DIR}")
  install(
    DIRECTORY ${_alibabacloud_credential_SOURCE_DIR}/include/
    DESTINATION ${INSTALL_INCLUDEDIR}
    COMPONENT dev
    FILES_MATCHING PATTERN "*.hpp"
  )
endif()

if(DEFINED _alibabacloud_open_api_v2_SOURCE_DIR)
  message(STATUS "Installing open-api-v2 headers from: ${_alibabacloud_open_api_v2_SOURCE_DIR}")
  install(
    DIRECTORY ${_alibabacloud_open_api_v2_SOURCE_DIR}/include/
    DESTINATION ${INSTALL_INCLUDEDIR}
    COMPONENT dev
    FILES_MATCHING PATTERN "*.hpp"
  )
endif()

# Install cmake config
install(
        EXPORT
        alibabacloud_sts20150401Targets
        NAMESPACE
        "alibabacloud_sts20150401::"
        DESTINATION
        ${INSTALL_CMAKEDIR}
        COMPONENT
        dev
)
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
)
configure_package_config_file(
        ${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
        INSTALL_DESTINATION ${INSTALL_CMAKEDIR}
)
install(
        FILES
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION
        ${INSTALL_CMAKEDIR}
)