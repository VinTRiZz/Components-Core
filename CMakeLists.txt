cmake_minimum_required(VERSION 3.7)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(
    COMPONENTS_COMPONENT_ROOT_DIR
    ${CMAKE_CURRENT_SOURCE_DIR}/..
)

# Component add common function
function(COMPONENTS_LINK_COMPONENT TARGET_NAME COMPONENT_NAME)
    if(NOT EXISTS ${COMPONENTS_COMPONENT_ROOT_DIR}/Components-${COMPONENT_NAME})
        message(FATAL_ERROR "Error searching for component: ${COMPONENT_NAME} (not exist)")
    endif()

    set(
        LIBRARY_INCLUDE_DIR
        ${COMPONENTS_COMPONENT_ROOT_DIR}/Components-${COMPONENT_NAME}/include
    )

    if(NOT EXISTS ${LIBRARY_INCLUDE_DIR})
        message(FATAL_ERROR "Error searching for ${COMPNAME} (not exist)")
    endif()

    target_link_libraries(
        ${TARGET_NAME} PRIVATE
        ${COMPNAME}
    )

    target_include_directories(
        ${TARGET_NAME} PUBLIC
        ${LIBRARY_INCLUDE_DIR}
    )
    message(STATUS "Linking target " ${TARGET_NAME} " : added lib " ${COMPNAME})
endfunction()



# Component configuration function
function(COMPONENTS_CONFIGURE_COMPONENT COMPNAME)
    # Check if project is self-hosted
    if (NOT DEFINED PROJECT_NAME)
        project(
            ${COMPNAME}
            VERSION 1.0.0
            LANGUAGES C CXX
        )
    endif()

    # Qt configuration
    set(CMAKE_AUTOUIC ON)
    set(CMAKE_AUTOMOC ON)
    set(CMAKE_AUTORCC ON)

    # Library target
    add_library(${COMPNAME} SHARED)
    target_link_libraries(${COMPNAME} PRIVATE ${TARGET_LINKED_LIBS} )

    LINK_COMPONENT(${COMPNAME} ${CMAKE_CURRENT_SOURCE_DIR}/.. Logger)

    # Files
    file(GLOB_RECURSE HEADERS "src/*.hpp" "src/*.h")
    file(GLOB_RECURSE SOURCES "src/*.cpp")
    target_sources(${COMPNAME} PRIVATE ${HEADERS} ${SOURCES} ${UIFILES})

    # Suffixes
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        target_compile_definitions(${COMPNAME} PRIVATE DEBUG_MODE)
        target_compile_options(${COMPNAME} PRIVATE -g)
        set_target_properties(${COMPNAME} PROPERTIES SUFFIX ".so.DEBUG")
    elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
        target_compile_definitions(${COMPNAME} PRIVATE NDEBUG)
        target_compile_options(${COMPNAME} PRIVATE -O3)
        set_target_properties(${COMPNAME} PROPERTIES SUFFIX ".so")
    endif()

    # Includes
    if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/src")
        message(FATAL_ERROR "Error configuring library! Not found sources dir: " "${CMAKE_CURRENT_SOURCE_DIR}/src")
    endif()

    target_include_directories(${COMPNAME} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/src")
    message(STATUS "Configured component: " ${COMPNAME})
endfunction()
