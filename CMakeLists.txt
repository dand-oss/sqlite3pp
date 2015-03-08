cmake_minimum_required(VERSION 2.8)

project(sqlite3pp CXX)

###############
# ASI default settings
list(APPEND CMAKE_MODULE_PATH "$ENV{ASV_CMAKE}")
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

include(find_library_list)
macro( find_3rdparty_lib lib_prefix lib_name)
  # note ARGN is all args past those expected as a list with ";" seperator
  find_libraries(${lib_prefix} ${ASV_PLAT_LIB} ${CMAKE_DEBUG_POSTFIX} SHARED ${lib_name})
endmacro(find_3rdparty_lib)

find_3rdparty_lib(SQLITE3     "sqlite3")

set( sqlite3pp_libs
    ${SQLITE3_LIBRARIES}
    )

set(libname sqlite3pp)

# do not remove - WG7 requires the static lib
set(staticlibname sqlite3ppstatic)

if (WIN32)
    add_library(${libname} SHARED ${s3_sources})
    target_link_libraries(${libname} ${sqlite3pp_libs})
    set_target_properties(${libname} PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
    install_static_pdb(${libname})

    add_library(${staticlibname} STATIC ${s3_sources})
    target_link_libraries(${staticlibname} ${sqlite3pp_libs})
    install_static_pdb(${staticlibname})
else(WIN32) # unix
    add_library(${libname} SHARED ${s3_sources})
    add_library(${staticlibname} STATIC ${s3_sources})
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
