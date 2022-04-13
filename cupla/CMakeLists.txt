cmake_minimum_required(VERSION 3.12 FATAL_ERROR)

project(bude_cupla)
SET(CMAKE_VERBOSE_MAKEFILE ON)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#====CUPLA-specific includes
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
find_package(cupla REQUIRED)
#===========================


set(SOURCES
        src/bude.cpp
        src/bude_cupla.cpp
	src/shared.cpp)

include_directories(src $CUPLA_INCLUDE_DIR)
CUPLA_ADD_EXECUTABLE(bude ${SOURCES})

separate_arguments(CXX_EXTRA_FLAGS)
separate_arguments(CXX_EXTRA_LINKER_FLAGS)

target_compile_options(bude
        PUBLIC
        -Wall
        -Wextra
        -Wcast-align
        -Wfatal-errors
        -Werror=return-type
        -Wno-unused-parameter
        -Wno-unused-variable
        -Wno-ignored-attributes

        ${EXTRA_FLAGS}
        )


if (NOT DEFINED NUM_TD_PER_THREAD)
    message(STATUS "NUM_TD_PER_THREAD unspecified, defaulting to 1")
    set(NUM_TD_PER_THREAD 1)
endif ()

if (NOT NUM_TD_PER_THREAD MATCHES "^[0-9]+$")
    message(FATAL_ERROR "NUM_TD_PER_THREAD must be an integer, got ${NUM_TD_PER_THREAD}")
endif ()

add_definitions(-DNUM_TD_PER_THREAD=${NUM_TD_PER_THREAD})


set(DEBUG_OPTIONS -O2 -fno-omit-frame-pointer ${CXX_EXTRA_FLAGS})
set(RELEASE_OPTIONS -Ofast -ffast-math -march=native -mtune=native ${CXX_EXTRA_FLAGS})

target_compile_options(bude PUBLIC "$<$<CONFIG:RelWithDebInfo>:${RELEASE_OPTIONS}>")
target_compile_options(bude PUBLIC "$<$<CONFIG:Release>:${RELEASE_OPTIONS}>")
target_compile_options(bude PUBLIC "$<$<CONFIG:Debug>:${DEBUG_OPTIONS}>")

if (${CMAKE_VERSION} VERSION_LESS "3.13.0")
    message(WARNING "target_link_options is only available in CMake >= 3.13.0, using fallback target_link_libraries, this may cause issues with some compilers")
    message(WARNING "whitespaces are not supported for CXX_EXTRA_LINKER_FLAGS/CXX_EXTRA_FLAGS in this mode as they are treated as libraries arguments (CMake splits them)")
    if (DEFINED CXX_EXTRA_LINKER_FLAGS)
        list(APPEND EXTRA_LINK_FLAGS "-Wl,${CXX_EXTRA_LINKER_FLAGS}")
    endif ()

    target_link_libraries(bude PUBLIC ${EXTRA_LINK_FLAGS})
    target_link_libraries(bude PUBLIC ${CXX_EXTRA_FLAGS})

else ()
    target_link_options(bude PUBLIC LINKER:${CXX_EXTRA_LINKER_FLAGS})
    target_link_options(bude PUBLIC ${CXX_EXTRA_FLAGS})
endif ()

