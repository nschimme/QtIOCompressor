cmake_minimum_required(VERSION 3.0.0)
project(QtSolutions_IOCompressor-2.3)

set(VERSION "2.3.1")

set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

find_package(Qt5Core REQUIRED)

# Use the compile definitions defined in the Qt 5 Core module
add_definitions(${Qt5Core_DEFINITIONS})

# Add the include directories for the Qt 5 Core module to the compile lines.
include_directories(${Qt5Core_INCLUDE_DIRS})

# Add compiler flags for building executables
if (CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
    add_definitions(-fPIC)
endif()

set(qtiocompressor_SRCS qtiocompressor.cpp)
set(qtiocompressor_HDRS qtiocompressor.h)

add_library(qtiocompressor SHARED ${qtiocompressor_SRCS})
add_library(qtiocompressorstatic STATIC ${qtiocompressor_SRCS})

find_package(ZLIB)
if (ZLIB_FOUND)
    message(STATUS "Found ZLIB: ${ZLIB_LIBRARIES} ${ZLIB_INCLUDE_DIRS}")
else()
    message(STATUS "Could not find local ZLIB; building local copy")
    include(ExternalProject)
    ExternalProject_Add(zlib
        GIT_REPOSITORY "https://github.com/madler/zlib.git"
        GIT_TAG "v1.2.11"
        SOURCE_DIR "${CMAKE_BINARY_DIR}/zlib-src"
        BINARY_DIR "${CMAKE_BINARY_DIR}/zlib-build"
        INSTALL_COMMAND ""
        CMAKE_ARGS "-DBUILD_SHARED_LIBS=ON"
        CMAKE_CACHE_ARGS "-DCMAKE_C_FLAGS:STRING=${CMAKE_C_FLAGS}"
        )
    if (WIN32)
        set(ZLIB_SHARED_LIBRARY_NAME zlib1)
        set(ZLIB_STATIC_LIBRARY_NAME zlibstatic)
    else()
        set(ZLIB_SHARED_LIBRARY_NAME z)
        set(ZLIB_STATIC_LIBRARY_NAME z)
    endif()
    set(ZLIB_SHARED_LIBRARY ${CMAKE_BINARY_DIR}/zlib-build/${CMAKE_SHARED_LIBRARY_PREFIX}${ZLIB_SHARED_LIBRARY_NAME}${CMAKE_SHARED_LIBRARY_SUFFIX})
    set(ZLIB_STATIC_LIBRARY ${CMAKE_BINARY_DIR}/zlib-build/${CMAKE_STATIC_LIBRARY_PREFIX}${ZLIB_STATIC_LIBRARY_NAME}${CMAKE_STATIC_LIBRARY_SUFFIX})
    set(ZLIB_LIBRARIES ${ZLIB_SHARED_LIBRARY} ${ZLIB_STATIC_LIBRARY})
    set(ZLIB_INCLUDE_DIRS ${CMAKE_BINARY_DIR}/zlib-src ${CMAKE_BINARY_DIR}/zlib-build)

    # Shared zlib Library
    add_dependencies(qtiocompressor zlib)

    # Static zlib Library
    add_library(zlibstatic STATIC IMPORTED)
    set_target_properties(zlibstatic PROPERTIES IMPORTED_LOCATION ${ZLIB_STATIC_LIBRARY})
    add_dependencies(zlibstatic zlib)
    add_dependencies(qtiocompressorstatic zlibstatic)
endif()

include_directories(${ZLIB_INCLUDE_DIRS})

target_link_libraries(qtiocompressor Qt5::Core ${ZLIB_LIBRARIES})
target_link_libraries(qtiocompressorstatic Qt5::Core ${ZLIB_LIBRARIES})

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    install(TARGETS qtiocompressor qtiocompressorstatic
        ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
        LIBRARY DESTINATION "${INSTALL_LIB_DIR}" )
endif()
if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL )
    install(FILES ${qtiocompressor_HDRS} DESTINATION "${INSTALL_INC_DIR}")
endif()

set_target_properties(qtiocompressor qtiocompressorstatic PROPERTIES OUTPUT_NAME QtSolutions_IOCompressor-2.3)
