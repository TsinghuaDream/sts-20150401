# alibabacloud_credential (credentials-cpp)
include(FetchContent)

# Try to use local source first (for development)
set(_LOCAL_CREDENTIALS "${CMAKE_CURRENT_LIST_DIR}/../../../credentials-cpp")
if(EXISTS "${_LOCAL_CREDENTIALS}/CMakeLists.txt")
    message(STATUS "Using local credentials-cpp source for alibabacloud_credential")
    FetchContent_Declare(
        _alibabacloud_credential
        SOURCE_DIR "${_LOCAL_CREDENTIALS}"
    )
else()
    # Try find_package first
    find_package(alibabacloud_credential QUIET)
    
    if(alibabacloud_credential_FOUND)
        message(STATUS "Found alibabacloud_credential version ${alibabacloud_credential_VERSION}")
        set(ALIBABACLOUD_CREDENTIAL_FOUND TRUE PARENT_SCOPE)
        return()
    endif()
    
    # Download from Git
    message(STATUS "alibabacloud_credential not found locally, downloading from GitHub...")
    FetchContent_Declare(
        _alibabacloud_credential
        GIT_REPOSITORY https://github.com/TsinghuaDream/credentials-cpp.git
        GIT_TAG        master
        GIT_SHALLOW    TRUE
    )
endif()

set(ENABLE_UNIT_TESTS OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(_alibabacloud_credential)

message(STATUS "  -> alibabacloud_credential ready: ${_alibabacloud_credential_SOURCE_DIR}")

if(TARGET ${PROJECT_NAME})
    target_include_directories(${PROJECT_NAME} PRIVATE "${_alibabacloud_credential_SOURCE_DIR}/include")
endif()

set(ALIBABACLOUD_CREDENTIAL_INCLUDE_DIR "${_alibabacloud_credential_SOURCE_DIR}/include" PARENT_SCOPE)
