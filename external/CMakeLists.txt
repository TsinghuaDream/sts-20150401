# ============================================================================
# External Dependencies
# ============================================================================
# Dependencies are loaded from subdirectories.
# Each subdirectory handles: local source fallback -> find_package -> FetchContent
# ============================================================================

message(STATUS "=== Configuring external dependencies ===")

# Disable tests for all dependencies
set(ENABLE_UNIT_TESTS OFF CACHE BOOL "" FORCE)
set(BUILD_UNIT_TESTS OFF CACHE BOOL "" FORCE)

# 1. darabonba_core (tea-cpp) - Core library
if(NOT TARGET darabonba_core)
    add_subdirectory(darabonba_core)
endif()

# 2. alibabacloud_credential (credentials-cpp)
if(NOT TARGET alibabacloud_credential)
    add_subdirectory(alibabacloud_credential)
endif()

# 3. alibabacloud_open_api_v2
if(NOT TARGET alibabacloud_open_api_v2)
    add_subdirectory(alibabacloud_open_api_v2)
endif()

message(STATUS "=== All dependencies configured ===")
