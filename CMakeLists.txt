cmake_minimum_required(VERSION 3.0)
project(libnoise LANGUAGES CXX)

# Define source directories
set(SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")

# Gather source files
file(GLOB ROOT_SOURCES "${SOURCE_DIR}/*.cpp")
file(GLOB MODEL_SOURCES "${SOURCE_DIR}/model/*.cpp")
file(GLOB MODULE_SOURCES "${SOURCE_DIR}/module/*.cpp")
set(NOISE_SOURCES_LIST ${ROOT_SOURCES} ${MODEL_SOURCES} ${MODULE_SOURCES})

# Gather header files
file(GLOB ROOT_HEADERS "${SOURCE_DIR}/*.h")
file(GLOB MODEL_HEADERS "${SOURCE_DIR}/model/*.h")
file(GLOB MODULE_HEADERS "${SOURCE_DIR}/module/*.h")
set(NOISE_HEADERS_LIST ${ROOT_HEADERS} ${MODEL_HEADERS} ${MODULE_HEADERS})

# Conditionally include Win32 files
set(PUBLIC_HEADERS ${ROOT_HEADERS})
if(WIN32)
    file(GLOB WIN32_SOURCES "${SOURCE_DIR}/win32/*.cpp")
    file(GLOB WIN32_HEADERS "${SOURCE_DIR}/win32/*.h")
    list(APPEND NOISE_SOURCES_LIST ${WIN32_SOURCES})
    list(APPEND NOISE_HEADERS_LIST ${WIN32_HEADERS})
    list(APPEND PUBLIC_HEADERS ${WIN32_HEADERS})
endif()

# Create static library
add_library(libnoise_static STATIC 
    ${NOISE_SOURCES_LIST} 
    ${NOISE_HEADERS_LIST}
)

# Set compiler definitions for GNU
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_definitions(libnoise_static PRIVATE _GNU_SOURCE)
endif()

# Configure target properties
set_target_properties(libnoise_static PROPERTIES
    OUTPUT_NAME libnoise_static
    PUBLIC_HEADER "${PUBLIC_HEADERS}"
)

# Modern include directory handling
target_include_directories(libnoise_static PUBLIC
    $<BUILD_INTERFACE:${SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Generate CMake alias target
add_library(noise::noise ALIAS libnoise_static)

# Installation configuration
include(GNUInstallDirs)
install(TARGETS libnoise_static
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libnoise
)

# Install subdirectory headers
install(FILES ${MODEL_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libnoise/model
)

install(FILES ${MODULE_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libnoise/module
)