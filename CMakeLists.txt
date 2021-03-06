cmake_minimum_required(VERSION 2.8)

project(sqlite3pp CXX)

###############
# ASI default settings
list(APPEND CMAKE_MODULE_PATH "$ENV{ASV_CMAKE}")
include(asv_cmake_defaults)
asv_cmake_defaults( )
###############

set(libname "sqlite3pp")
# do not remove - WG 7.0 requires the static lib
set(staticlibname "sqlite3pp-static")

# We built SQLite as cmake package
find_package(Threads REQUIRED)
find_package(SQLite3 CONFIG)

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

set(sources
    src/sqlite3pp.cpp
    src/sqlite3pp.h
    src/sqlite3ppext.cpp
    src/sqlite3ppext.h
    )

add_library(${libname} SHARED)
target_sources(${libname} PRIVATE ${sources})
target_link_libraries(${libname} PUBLIC
    SQLite::SQLite3-shared
    )

add_library(${staticlibname} STATIC)
target_sources(${staticlibname} PRIVATE ${sources})
target_link_libraries(${staticlibname} PUBLIC
    SQLite::SQLite3-shared # needed for includes
    )

if (WIN32)
    set_target_properties(${libname} PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
    install_static_pdb(${staticlibname})
endif(WIN32)

# copy the lib
install(TARGETS ${libname}
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    )

install(TARGETS ${staticlibname}
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    )

# copy the h files
install(
    FILES src/sqlite3pp.h src/sqlite3ppext.h
    DESTINATION include
    )
