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
    message(STATUS "Could not find local ZLIB, building external copy")
    include(ExternalProject)
    ExternalProject_Add(zlib
        GIT_REPOSITORY "https://github.com/madler/zlib.git"
        SOURCE_DIR "${CMAKE_BINARY_DIR}/zlib-src"
        BINARY_DIR "${CMAKE_BINARY_DIR}/zlib-build"
        INSTALL_COMMAND ""
        CMAKE_CACHE_ARGS "-DCMAKE_C_FLAGS:STRING=${CMAKE_C_FLAGS}"
        )
    set(ZLIB_LIBRARIES ${CMAKE_BINARY_DIR}/zlib-build/${CMAKE_SHARED_LIBRARY_PREFIX}z${CMAKE_SHARED_LIBRARY_SUFFIX}
                       ${CMAKE_BINARY_DIR}/zlib-build/${CMAKE_STATIC_LIBRARY_PREFIX}z${CMAKE_STATIC_LIBRARY_SUFFIX})
    set(ZLIB_SRC_DIR ${CMAKE_BINARY_DIR}/zlib-src)
    set(ZLIB_INCLUDE_DIRS ${ZLIB_SRC_DIR} ${CMAKE_BINARY_DIR}/zlib-build)
    add_dependencies(qtiocompressor zlib)
    message(STATUS "Will build ZLIB: ${ZLIB_LIBRARIES}")
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
