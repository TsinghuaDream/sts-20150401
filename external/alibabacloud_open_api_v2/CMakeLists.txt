# alibabacloud_open_api_v2
include(FetchContent)

# Try to use local source first (for development)
set(_LOCAL_OPEN_API "${CMAKE_CURRENT_LIST_DIR}/../../../alibabacloud-open-api-v2")
if(EXISTS "${_LOCAL_OPEN_API}/CMakeLists.txt")
    message(STATUS "Using local alibabacloud-open-api-v2 source")
    FetchContent_Declare(
        _alibabacloud_open_api_v2
        SOURCE_DIR "${_LOCAL_OPEN_API}"
    )
else()
    # Try find_package first
    find_package(alibabacloud_open_api_v2 QUIET)
    
    if(alibabacloud_open_api_v2_FOUND)
        message(STATUS "Found alibabacloud_open_api_v2 version ${alibabacloud_open_api_v2_VERSION}")
        set(ALIBABACLOUD_OPEN_API_V2_FOUND TRUE PARENT_SCOPE)
        return()
    endif()
    
    # Download from Git
    message(STATUS "alibabacloud_open_api_v2 not found locally, downloading from GitHub...")
    FetchContent_Declare(
        _alibabacloud_open_api_v2
        GIT_REPOSITORY https://github.com/TsinghuaDream/alibabacloud-open-api-v2.git
        GIT_TAG        master
        GIT_SHALLOW    TRUE
    )
endif()

set(ENABLE_UNIT_TESTS OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(_alibabacloud_open_api_v2)

message(STATUS "  -> alibabacloud_open_api_v2 ready: ${_alibabacloud_open_api_v2_SOURCE_DIR}")

if(TARGET ${PROJECT_NAME})
    target_include_directories(${PROJECT_NAME} PRIVATE "${_alibabacloud_open_api_v2_SOURCE_DIR}/include")
endif()

set(ALIBABACLOUD_OPEN_API_V2_INCLUDE_DIR "${_alibabacloud_open_api_v2_SOURCE_DIR}/include" PARENT_SCOPE)
