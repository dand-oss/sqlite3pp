cmake_minimum_required(VERSION 3.31)

project(sqlite3pp
    LANGUAGES CXX
    VERSION 1.1.0
    DESCRIPTION "C++ wrapper for SQLite3"
)

###############
# ASI default settings
list(APPEND CMAKE_MODULE_PATH "$ENV{ASV_CMAKE}")
include(asv_cmake_defaults)
asv_cmake_defaults()
###############

set(libname "sqlite3pp")
# do not remove - WG 7.0 requires the static lib
set(staticlibname "sqlite3pp-static")

# We built SQLite as cmake package
find_package(Threads REQUIRED)
find_package(SQLite3 CONFIG REQUIRED)

asv_print_value(
    CMAKE_PREFIX_PATH
    CMAKE_INSTALL_PREFIX
    CMAKE_LIBRARY_PATH
    CMAKE_FIND_LIBRARY_PREFIXES
    CMAKE_FIND_LIBRARY_SUFFIXES
    ASV_PLAT_PORTS
    ASV_PLAT_INC
    ASV_PLAT_LIB
)

asv_print_package_properties(
    SQLite::SQLite3-shared
)

# -----------------------
# Public header list
# -----------------------
set(public_headers
    src/sqlite3pp.h
    src/sqlite3ppext.h
)

# -----------------------
# Shared library
# -----------------------
add_library(${libname} SHARED)

# Use file sets for public headers
target_sources(${libname} PUBLIC
    FILE_SET HEADERS
    BASE_DIRS src
    FILES ${public_headers}
)

target_sources(${libname} PRIVATE
    src/sqlite3pp.cpp
    src/sqlite3ppext.cpp
)

target_include_directories(${libname} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(${libname} PUBLIC
    SQLite::SQLite3-shared
)

# -----------------------
# Static library
# -----------------------
add_library(${staticlibname} STATIC)

# Use file sets for public headers (shared between both targets)
target_sources(${staticlibname} PUBLIC
    FILE_SET HEADERS
    BASE_DIRS src
    FILES ${public_headers}
)

target_sources(${staticlibname} PRIVATE
    src/sqlite3pp.cpp
    src/sqlite3ppext.cpp
)

target_include_directories(${staticlibname} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(${staticlibname} PUBLIC
    SQLite::SQLite3  # needed for includes
)

# -----------------------
# Platform-specific settings
# -----------------------
if (WIN32)
    set_target_properties(${libname} PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
    install_static_pdb(${staticlibname})
endif()

# -----------------------
# Install libraries and headers
# -----------------------
install(TARGETS ${libname} ${staticlibname}
    EXPORT ${PROJECT_NAME}Targets
    FILE_SET HEADERS DESTINATION include
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

# Install export file
install(EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION lib/cmake/${PROJECT_NAME}
)

# Generate and install CMake config file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    @ONLY
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION lib/cmake/${PROJECT_NAME}
)

# CPack configuration for packaging
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_DESCRIPTION "C++ wrapper for SQLite3")
set(CPACK_PACKAGE_VENDOR "xAI")
set(CPACK_GENERATOR "ZIP;TGZ")
include(CPack)
