cmake_minimum_required(VERSION 3.20)

# find vcpkg for Windows
if (DEFINED ENV{VCPKG_ROOT} AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    file(TO_CMAKE_PATH "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake" VCPKG_CMAKE_TOOLCHAIN_FILE)
    if (EXISTS ${VCPKG_CMAKE_TOOLCHAIN_FILE})
        message(STATUS "Using vcpkg toolchain file: ${VCPKG_CMAKE_TOOLCHAIN_FILE}")
        set(CMAKE_TOOLCHAIN_FILE ${VCPKG_CMAKE_TOOLCHAIN_FILE} CACHE FILEPATH "" FORCE)
    endif()
endif()

project(binapi)

option(BUILD_EXAMPLES "Build examples" OFF)

# set compile options
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")

if(UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
endif()
if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}  /W4 /bigobj")
endif()

# find packages
find_package(Boost REQUIRED)
find_package(ZLIB REQUIRED)
find_package(OpenSSL REQUIRED)

# defines
add_definitions(
    -UNDEBUG
    -DDTF_HEADER_ONLY
)

include_directories(
    ./include
)

set(BINAPI_HEADERS
    binapi/api.hpp
    binapi/flatjson.hpp
    binapi/dtf.hpp
    binapi/double_type.hpp
    binapi/enums.hpp
    binapi/errors.hpp
    binapi/invoker.hpp
    binapi/message.hpp
    binapi/pairslist.hpp
    binapi/reports.hpp
    binapi/tools.hpp
    binapi/types.hpp
    binapi/websocket.hpp
)

set(BINAPI_SOURCES
    src/api.cpp
    src/enums.cpp
    src/errors.cpp
    src/pairslist.cpp
    src/reports.cpp
    src/tools.cpp
    src/types.cpp
    src/websocket.cpp
)

# create library, to use externaly link BinAPI::binapilib
add_library(binapilib STATIC ${BINAPI_SOURCES})
add_library(BinAPI::binapilib ALIAS binapilib)

# bind include files to target
target_include_directories(binapilib PUBLIC ${BINAPI_HEADERS})

# link libraries to target
target_link_libraries(
    binapilib
    ZLIB::ZLIB
    OpenSSL::SSL
    OpenSSL::Crypto
    Boost::boost
)

if (DEFINED ${BOOST_LIB_DIR})
    target_link_libraries(
        binapilib
        -L${BOOST_LIB_DIR}
    )
endif()

# build executable
add_executable(
    ${PROJECT_NAME}
    #
    main.cpp
)
target_link_libraries(${PROJECT_NAME} BinAPI::binapilib)

# Check if the BUILD_EXAMPLES option is set
if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()
