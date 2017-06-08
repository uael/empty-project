cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)
project(foo)

cmake_policy(SET CMP0054 OLD)
cmake_policy(SET CMP0045 OLD)

if ("${CMAKE_C_COMPILER_ID}" MATCHES "Clang")
    set(CLANG TRUE)
elseif ("${CMAKE_C_COMPILER_ID}" MATCHES "GNU")
    set(GCC TRUE)
elseif ("${CMAKE_C_COMPILER_ID}" MATCHES "Intel")
    set(ICC TRUE)
elseif(NOT MSVC)
    message(FATAL_ERROR "Unknown compiler")
endif()

if(CMAKE_VERSION VERSION_LESS "3.1")
    if(GCC)
        set(CMAKE_C_FLAGS "-std=c99 ${CMAKE_C_FLAGS}")
    endif()
else()
    set(CMAKE_C_STANDARD 99)
endif()

option(COVERAGE "Turn on COVERAGE support" OFF)

if(COVERAGE AND NOT MSVC)
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} --coverage")
endif()

set(project_HEADERS)
set(project_SOURCES)
set(project_INCLUDE_DIR ${CMAKE_CURRENT_LIST_DIR}/include)
set(project_SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/src)
set(project_TEST_DIR ${CMAKE_CURRENT_LIST_DIR}/test)
set(project_VENDOR_DIR ${CMAKE_CURRENT_LIST_DIR}/vendor)

if(EXISTS ${project_INCLUDE_DIR})
    file(GLOB_RECURSE project_HEADERS ${project_HEADERS} ${project_INCLUDE_DIR}/${PROJECT_NAME}/*.h)
endif()
if(EXISTS ${project_SOURCE_DIR})
    file(GLOB_RECURSE project_SOURCES ${project_SOURCES} ${project_SOURCE_DIR}/*.c)
endif()

add_library(${PROJECT_NAME} STATIC ${project_SOURCES} ${project_HEADERS})
set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${project_HEADERS}")

if(EXISTS ${project_INCLUDE_DIR})
    target_include_directories(${PROJECT_NAME} PUBLIC ${project_INCLUDE_DIR})
endif()
if(EXISTS ${project_SOURCE_DIR})
    target_include_directories(${PROJECT_NAME} PRIVATE ${project_SOURCE_DIR})
endif()

if(MSVC)
    target_compile_options(${PROJECT_NAME}
        PRIVATE /Oy
        PRIVATE /O$<$<CONFIG:Debug>:d>$<$<CONFIG:Release>:x>
    )
else()
    target_compile_options(${PROJECT_NAME}
        PRIVATE -Wall -Werror -Wextra -fomit-frame-pointer
        PRIVATE -O$<$<CONFIG:Debug>:0 -g3>$<$<CONFIG:Release>:3>
    )
endif()

if(MSVC)
    set(CMAKE_FLAGS
        CMAKE_CXX_FLAGS
        CMAKE_CXX_FLAGS_DEBUG
        CMAKE_CXX_FLAGS_RELEASE
        CMAKE_C_FLAGS
        CMAKE_C_FLAGS_DEBUG
        CMAKE_C_FLAGS_RELEASE
    )
    foreach(CMAKE_FLAG ${CMAKE_FLAGS})
        string(REPLACE "/MD" "/MT" ${CMAKE_FLAG} "${${CMAKE_FLAG}}")
        string(REPLACE "/MDd" "/MTd" ${CMAKE_FLAG} "${${CMAKE_FLAG}}")
    endforeach()
endif()

if(EXISTS ${project_VENDOR_DIR}/CMakeLists.txt)
    add_subdirectory(${project_VENDOR_DIR})
endif()

if(EXISTS ${project_TEST_DIR})
    enable_testing()
    if(EXISTS ${project_VENDOR_DIR}/cute)
        add_subdirectory(${project_VENDOR_DIR}/cute)
    endif()
    add_subdirectory(${project_TEST_DIR})
endif()

install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    PUBLIC_HEADER DESTINATION include/${PROJECT_NAME}
)
if(EXISTS ${project_INCLUDE_DIR}/${PROJECT_NAME}.h)
    install(FILES ${project_INCLUDE_DIR}/${PROJECT_NAME}.h
        DESTINATION include
    )
endif()

set(project_HEADERS)
set(project_SOURCES)
set(project_INCLUDE_DIR)
set(project_SOURCE_DIR)
set(project_TEST_DIR)
set(project_VENDOR_DIR)
