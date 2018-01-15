CMAKE_MINIMUM_REQUIRED(VERSION 3.0.0)
PROJECT(QtSolutions_IOCompressor-2.3)

SET(VERSION "1.0.0")

SET(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
SET(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")

OPTION(SKIP_INSTALL_ALL "Skip installing everything" FALSE)
OPTION(SKIP_INSTALL_LIBRARIES "Skip installing libraries" FALSE)
OPTION(SKIP_INSTALL_HEADERS "Skip installing headers" FALSE)

SET(CMAKE_INCLUDE_CURRENT_DIR ON)
SET(CMAKE_AUTOMOC ON)

FIND_PACKAGE(Qt5Core REQUIRED)

# Use the compile definitions defined in the Qt 5 Core module
ADD_DEFINITIONS(${Qt5Core_DEFINITIONS})

# Add the include directories for the Qt 5 Core module to the compile lines.
INCLUDE_DIRECTORIES(${Qt5Core_INCLUDE_DIRS})

# Add compiler flags for building executables
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
ADD_DEFINITIONS(-fPIC)

SET(qtiocompressor_SRCS src/qtiocompressor.cpp)

ADD_LIBRARY(qtiocompressor SHARED ${qtiocompressor_SRCS})
ADD_LIBRARY(qtiocompressorobj OBJECT ${qtiocompressor_SRCS})
ADD_LIBRARY(qtiocompressorstatic STATIC ${qtiocompressor_SRCS})

FIND_PACKAGE(ZLIB)
IF(ZLIB_FOUND)
    MESSAGE(STATUS "Found ZLIB: ${ZLIB_LIBRARIES} ${ZLIB_INCLUDE_DIRS}")
else()
    MESSAGE(STATUS "Could not find local ZLIB; building local copy")
    INCLUDE(ExternalProject)
    ExternalProject_Add(zlib
        GIT_REPOSITORY "https://github.com/madler/zlib.git"
        GIT_TAG "v1.2.11"
        SOURCE_DIR "${CMAKE_BINARY_DIR}/zlib-src"
        BINARY_DIR "${CMAKE_BINARY_DIR}/zlib-build"
        INSTALL_COMMAND ""
        CMAKE_ARGS "-DBUILD_SHARED_LIBS=ON"
        CMAKE_CACHE_ARGS "-DCMAKE_C_FLAGS:STRING=${CMAKE_C_FLAGS}"
        )
    IF(WIN32)
        SET(ZLIB_SHARED_LIBRARY_NAME zlib1)
        SET(ZLIB_STATIC_LIBRARY_NAME zlibstatic)
    else()
        SET(ZLIB_SHARED_LIBRARY_NAME z)
        SET(ZLIB_STATIC_LIBRARY_NAME z)
    ENDIF()
    SET(ZLIB_SHARED_LIBRARY ${CMAKE_BINARY_DIR}/zlib-build/${CMAKE_SHARED_LIBRARY_PREFIX}${ZLIB_SHARED_LIBRARY_NAME}${CMAKE_SHARED_LIBRARY_SUFFIX})
    SET(ZLIB_STATIC_LIBRARY ${CMAKE_BINARY_DIR}/zlib-build/${CMAKE_STATIC_LIBRARY_PREFIX}${ZLIB_STATIC_LIBRARY_NAME}${CMAKE_STATIC_LIBRARY_SUFFIX})
    SET(ZLIB_LIBRARIES ${ZLIB_SHARED_LIBRARY} ${ZLIB_STATIC_LIBRARY})
    SET(ZLIB_INCLUDE_DIRS ${CMAKE_BINARY_DIR}/zlib-src ${CMAKE_BINARY_DIR}/zlib-build)

    # Shared zlib Library
    ADD_DEPENDENCIES(qtiocompressor zlib)
    ADD_DEPENDENCIES(qtiocompressorobj zlib)

    # Static zlib Library
    ADD_LIBRARY(zlibstatic STATIC IMPORTED)
    SET_TARGET_PROPERTIES(zlibstatic PROPERTIES IMPORTED_LOCATION ${ZLIB_STATIC_LIBRARY})
    ADD_DEPENDENCIES(zlibstatic zlib)
    ADD_DEPENDENCIES(qtiocompressorstatic zlibstatic)
ENDIF()

INCLUDE_DIRECTORIES(${ZLIB_INCLUDE_DIRS})

TARGET_LINK_LIBRARIES(qtiocompressor Qt5::Core ${ZLIB_LIBRARIES})
TARGET_LINK_LIBRARIES(qtiocompressorstatic Qt5::Core ${ZLIB_LIBRARIES})

SET_TARGET_PROPERTIES(qtiocompressor qtiocompressorstatic PROPERTIES OUTPUT_NAME QtSolutions_IOCompressor-2.3)

IF(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    INSTALL(TARGETS qtiocompressor qtiocompressorstatic
        ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
        LIBRARY DESTINATION "${INSTALL_LIB_DIR}" )
ENDIF()
IF(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL )
    INSTALL(FILES ${qtiocompressor_HDRS} DESTINATION "${INSTALL_INC_DIR}")
ENDIF()
