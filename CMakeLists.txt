cmake_minimum_required(VERSION 3.7)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if (NOT COMPONENTS_COMPONENT_ROOT_DIR)
    set(
        COMPONENTS_COMPONENT_ROOT_DIR
        ${CMAKE_CURRENT_SOURCE_DIR}/..
    )

else()
    message(STATUS "[COMPONENTS] Components root dir set as: " ${COMPONENTS_COMPONENT_ROOT_DIR})
endif()

if (NOT COMPONENTS_INSTALL_PATH)
    set(
        COMPONENTS_INSTALL_PATH
        ${CMAKE_CURRENT_SOURCE_DIR}/../BIN
    )
else()
    message(STATUS "[COMPONENTS] Components install prefix set as: " ${COMPONENTS_INSTALL_PATH})
endif()

# ARGUMENTS:
# COMPONENT_NAME --> Name of linking component
# ARGS AFTER COMPONENT_NAME --> Extra libraries to link (boost_system, ssl, etc.)
# Component add common function
function(COMPONENTS_LINK_COMPONENT TARGET_NAME COMPONENT_NAME)
    if(NOT EXISTS ${COMPONENTS_COMPONENT_ROOT_DIR}/Components-${COMPONENT_NAME})
        message(FATAL_ERROR "[COMPONENTS] Error searching for: ${COMPONENT_NAME} (not exist). Searching directory: ${COMPONENTS_COMPONENT_ROOT_DIR}")
    endif()

    set(
        LIBRARY_INCLUDE_DIR
        ${COMPONENTS_COMPONENT_ROOT_DIR}/Components-${COMPONENT_NAME}/include
    )

    if(NOT EXISTS ${LIBRARY_INCLUDE_DIR})
        message(FATAL_ERROR "[COMPONENTS] Error searching for ${COMPONENT_NAME} (not exist). Searching directory: ${COMPONENTS_COMPONENT_ROOT_DIR}")
    endif()

    target_link_libraries(
        ${TARGET_NAME} PRIVATE
        ${COMPONENT_NAME}
    )

    target_include_directories(
        ${TARGET_NAME} PUBLIC
        ${LIBRARY_INCLUDE_DIR}
    )
    message(STATUS "[COMPONENTS] Linking target " ${TARGET_NAME} " : added component " ${COMPONENT_NAME})
endfunction()



# Component configuration function
function(COMPONENTS_CONFIGURE_COMPONENT COMPONENT_NAME)
    # Check if project is self-hosted
    if (NOT DEFINED PROJECT_NAME)
        project(
            ${COMPONENT_NAME}
            VERSION 1.0.0
            LANGUAGES C CXX
        )
    endif()

    # Qt configuration
    set(CMAKE_AUTOUIC ON)
    set(CMAKE_AUTOMOC ON)
    set(CMAKE_AUTORCC ON)

    # Library target
    add_library(${COMPONENT_NAME} SHARED)
    target_link_libraries(${COMPONENT_NAME} PRIVATE ${ARGN} )

    install(TARGETS ${COMPONENT_NAME}
        RUNTIME     DESTINATION ${COMPONENTS_INSTALL_PATH}
        LIBRARY     DESTINATION ${COMPONENTS_INSTALL_PATH}/lib
        ARCHIVE     DESTINATION ${COMPONENTS_INSTALL_PATH}/lib/static
        INCLUDES    DESTINATION ${COMPONENTS_INSTALL_PATH}/include
    )

    # Files
    file(GLOB_RECURSE HEADERS "src/*.hpp" "src/*.h")
    file(GLOB_RECURSE SOURCES "src/*.cpp")
    target_sources(${COMPONENT_NAME} PRIVATE ${HEADERS} ${SOURCES} ${UIFILES})

    # Suffixes
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        target_compile_definitions(${COMPONENT_NAME} PRIVATE DEBUG_MODE)
        target_compile_options(${COMPONENT_NAME} PRIVATE -g)
        set_target_properties(${COMPONENT_NAME} PROPERTIES SUFFIX ".so.DEBUG")
    elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
        target_compile_definitions(${COMPONENT_NAME} PRIVATE NDEBUG)
        target_compile_options(${COMPONENT_NAME} PRIVATE -O3)
        set_target_properties(${COMPONENT_NAME} PROPERTIES SUFFIX ".so")
    endif()

    # Includes
    if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/src")
        message(FATAL_ERROR "[COMPONENTS] Error configuring component! Not found sources dir: " "${CMAKE_CURRENT_SOURCE_DIR}/src")
    endif()

    target_include_directories(${COMPONENT_NAME} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/include")
    message(STATUS "[COMPONENTS] Configured component: " ${COMPONENT_NAME})
endfunction()


# Qt component searching function
function(COMPONENTS_REQUIRE_QT COMPS_QT_VER)
    find_package(QT NAMES Qt${COMPS_QT_VER} REQUIRED COMPONENTS ${ARGN})
    find_package(Qt${COMPS_QT_VER} REQUIRED COMPONENTS ${ARGN})
endfunction()
