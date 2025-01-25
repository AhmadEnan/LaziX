cmake_minimum_required(VERSION 3.15)
project(LaziX)  # This names the solution

# Set C++ standard for all targets
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Add library project
add_library(lazix STATIC
    lazix/include/lazix/lazix.h    # Public header
    lazix/src/lazix.cpp            # Implementation
)

# Public include directory - exposed to users of the library
target_include_directories(lazix PUBLIC 
    ${CMAKE_CURRENT_SOURCE_DIR}/lazix/include
)

# Private include directory - for library's own implementation
target_include_directories(lazix PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/lazix/src
)

# Configuration-specific settings
target_compile_definitions(lazix PRIVATE
    $<$<CONFIG:Debug>:LAZIX_DEBUG>
)

# Add executable project
add_executable(Sandbox
    sandbox/src/main.cpp
)

if(MSVC)
    # 1. Explicitly set startup project
    set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT Sandbox)
    
    # 2. Force executable output path (critical for VS debugging)
    set_target_properties(Sandbox PROPERTIES
        VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/bin/$<CONFIG>"
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/$<CONFIG>"
    )
endif()

# Link library to executable and set build order
target_link_libraries(Sandbox PRIVATE lazix)
add_dependencies(Sandbox lazix)  # Ensure build order

# Set output directories (optional but recommended)
set_target_properties(lazix PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
)

set_target_properties(Sandbox PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_target_properties(lazix Sandbox PROPERTIES FOLDER "Projects")