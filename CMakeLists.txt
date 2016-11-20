cmake_minimum_required(VERSION 2.8)

project(sqlite3pp CXX)

###############
# ASI default settings
list(APPEND CMAKE_MODULE_PATH "$ENV{ASI}/cmake/Modules")
include(asv_cmake_defaults)
asv_cmake_defaults( )
###############

# where our sqlite build lives
include_directories(AFTER ${ASV_PLAT_INC})
message(STATUS "include_directory=\"${ASV_PLAT_INC}\"")

foreach(s
    sqlite3pp.cpp     sqlite3ppext.cpp
    )
  list(APPEND s3_sources src/${s})
endforeach()

if (WIN32)
    add_library(sqlite3pp STATIC ${s3_sources})
    install_static_pdb(sqlite3pp) # ASV
else(WIN32)
    add_library(sqlite3pp SHARED ${s3_sources})
endif(WIN32)

# copy the lib
install(TARGETS sqlite3pp
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    )

# copy the h files
install(
    FILES src/sqlite3pp.h src/sqlite3ppext.h
    DESTINATION include
    )
