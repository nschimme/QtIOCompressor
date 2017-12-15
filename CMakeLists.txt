cmake_minimum_required(VERSION 3.0.0)
project(qtiocompressor)

set(VERSION "2.3.1")

set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC OFF)

find_package(Qt5Core REQUIRED)

# Try to find system copy of zlib
find_package(ZLIB)
if (ZLIB_INCLUDE_DIRS AND ZLIB_LIBRARIES)
    message(STATUS "Found ZLIB: ${ZLIB_LIBRARIES} ${ZLIB_INCLUDE_DIRS}")
else()
    include(ExternalProject)
    message(STATUS "Could not find local ZLIB, building bundled copy")
    ExternalProject_Add(
      ZLIB
      GIT_REPOSITORY "https://github.com/madler/zlib.git"
      GIT_TAG "master"
      SOURCE_DIR "${CMAKE_BINARY_DIR}/zlib-src"
      BINARY_DIR "${CMAKE_BINARY_DIR}/zlib-build"
      CONFIGURE_COMMAND ""
      BUILD_COMMAND     ""
      INSTALL_COMMAND ""
      TEST_COMMAND ""
    )
endif()

# Add the include directories for the Qt 5 Core module to the compile lines.
include_directories(${Qt5Core_INCLUDE_DIRS})

# Use the compile definitions defined in the Qt 5 Core module
add_definitions(${Qt5Core_DEFINITIONS})

# Add compiler flags for building executables
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
add_definitions(-fPIC)

set(qtiocompressor_SRCS qtiocompressor.cpp)
set(qtiocompressor_HDRS qtiocompressor.h)

QT5_WRAP_CPP(qtiocompressor_MOC_SRCS ${qtiocompressor_HDRS})

add_library(qtiocompressor SHARED ${qtiocompressor_SRCS} ${qtiocompressor_MOC_SRCS})
add_library(qtiocompressorstatic STATIC ${qtiocompressor_SRCS} ${qtiocompressor_MOC_SRCS})

target_link_libraries(qtiocompressor Qt5::Core ${ZLIB_LIBRARIES})

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    install(TARGETS qtiocompressor qtiocompressorstatic
        ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
        LIBRARY DESTINATION "${INSTALL_LIB_DIR}" )
endif()
if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL )
    install(FILES ${qtiocompressor_HDRS} DESTINATION "${INSTALL_INC_DIR}")
endif()
