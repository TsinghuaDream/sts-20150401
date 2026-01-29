# darabonba_core (tea-cpp) - Core library
include(FetchContent)

# Try to use local source first (for development)
set(_LOCAL_TEA_CPP "${CMAKE_CURRENT_LIST_DIR}/../../../tea-cpp")
if(EXISTS "${_LOCAL_TEA_CPP}/CMakeLists.txt")
    message(STATUS "Using local tea-cpp source for darabonba_core")
    FetchContent_Declare(
        _darabonba_core
        SOURCE_DIR "${_LOCAL_TEA_CPP}"
    )
else()
    # Try find_package first
    find_package(darabonba_core QUIET)
    
    if(darabonba_core_FOUND)
        message(STATUS "Found darabonba_core version ${darabonba_core_VERSION}")
        set(DARABONBA_CORE_FOUND TRUE PARENT_SCOPE)
        return()
    endif()
    
    # Download from Git
    message(STATUS "darabonba_core not found locally, downloading from GitHub...")
    FetchContent_Declare(
        _darabonba_core
        GIT_REPOSITORY https://github.com/TsinghuaDream/tea-cpp.git
        GIT_TAG        master
        GIT_SHALLOW    TRUE
    )
endif()

set(ENABLE_UNIT_TESTS OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(_darabonba_core)

# Linux: link uuid library
if(UNIX AND NOT APPLE)
    cmake_policy(SET CMP0079 NEW)
    find_package(PkgConfig QUIET)
    if(PkgConfig_FOUND)
        pkg_check_modules(UUID uuid)
        if(UUID_FOUND AND TARGET darabonba_core)
            target_link_libraries(darabonba_core ${UUID_LIBRARIES})
            target_include_directories(darabonba_core PUBLIC ${UUID_INCLUDE_DIRS})
        elseif(TARGET darabonba_core)
            target_link_libraries(darabonba_core uuid)
        endif()
    elseif(TARGET darabonba_core)
        target_link_libraries(darabonba_core uuid)
    endif()
endif()

message(STATUS "  -> darabonba_core ready: ${_darabonba_core_SOURCE_DIR}")

if(TARGET ${PROJECT_NAME})
    target_include_directories(${PROJECT_NAME} PRIVATE "${_darabonba_core_SOURCE_DIR}/include")
endif()

set(DARABONBA_CORE_INCLUDE_DIR "${_darabonba_core_SOURCE_DIR}/include" PARENT_SCOPE)
