cmake_minimum_required(VERSION 3.16)
project(ZenohSubscriber C)

# Force C11 standard
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# -----------------------------------------------------------------------------
# Find Zenoh-C Dependencies
# -----------------------------------------------------------------------------
# Pixi usually populates CMAKE_PREFIX_PATH, but we'll search for the headers/libs
if(DEFINED ENV{CONDA_PREFIX})
    set(PIXI_INCLUDE_DIR "$ENV{CONDA_PREFIX}/include")
    set(PIXI_LIB_DIR "$ENV{CONDA_PREFIX}/lib")
else()
    message(WARNING "CONDA_PREFIX environment variable not found. Are you running inside 'pixi run'?")
endif()

# Explicitly hunt for zenoh-pico headers in likely subdirectories
find_path(ZENOHPICO_INCLUDE_DIR 
    NAMES zenohpico.h zenoh-pico.h
    PATHS ${PIXI_INCLUDE_DIR}
    PATH_SUFFIXES zenoh-pico zenohpico
)

# Hunt for the library file (libzenohpico.so / libzenohpico.dylib)
find_library(ZENOHPICO_LIBRARY 
    NAMES zenohpico
    PATHS ${PIXI_LIB_DIR}
)

# Debugging logs so you can see exactly where CMake is checking
message(STATUS "Searching Pixi root: $ENV{CONDA_PREFIX}")
message(STATUS "Found Zenoh-Pico Header at: ${ZENOHPICO_INCLUDE_DIR}")
message(STATUS "Found Zenoh-Pico Library at: ${ZENOHPICO_LIBRARY}")

if(NOT ZENOHPICO_INCLUDE_DIR OR NOT ZENOHPICO_LIBRARY)
    message(FATAL_ERROR "Still missing Zenoh-Pico! Double check 'pixi list' to see if zenoh-pico is actually installed.")
endif()

# -----------------------------------------------------------------------------
# Target Configuration
# -----------------------------------------------------------------------------
# Define the executable and its source file
add_executable(z_sub src/z_sub.c)
add_executable(z_sub_tsl src/z_sub_tsl.c)
add_executable(z_pub src/z_pub.c)

# Include directories
target_include_directories(z_sub PRIVATE ${ZENOHPICO_INCLUDE_DIR})
target_include_directories(z_sub_tsl PRIVATE ${ZENOHPICO_INCLUDE_DIR})
target_include_directories(z_pub PRIVATE ${ZENOHPICO_INCLUDE_DIR})

# Link libraries
target_link_libraries(z_sub PRIVATE ${ZENOHPICO_LIBRARY})
target_link_libraries(z_sub_tsl PRIVATE ${ZENOHPICO_LIBRARY})
target_link_libraries(z_pub PRIVATE ${ZENOHPICO_LIBRARY})

# Unix-specific compiler flags and system libraries
if(UNIX)
    target_compile_options(z_sub PRIVATE -Wall -Wextra -O2)
    target_compile_options(z_sub_tsl PRIVATE -Wall -Wextra -O2)
    target_compile_options(z_pub PRIVATE -Wall -Wextra -O2)
    
    # If on Linux, Zenoh might require pthread and math libraries
    if(CMAKE_SYSTEM_NAME MATCHES "Linux")
        target_compile_definitions(z_sub PRIVATE ZENOH_LINUX)
        target_compile_definitions(z_sub_tsl PRIVATE ZENOH_LINUX)
        target_compile_definitions(z_pub PRIVATE ZENOH_LINUX)
        target_compile_definitions(z_sub PRIVATE Z_FEATURE_LINK_TLS)
        target_compile_definitions(z_sub_tsl PRIVATE Z_FEATURE_LINK_TLS)
        target_compile_definitions(z_pub PRIVATE Z_FEATURE_LINK_TLS)
        target_link_libraries(z_sub PRIVATE pthread m)
        target_link_libraries(z_sub_tsl PRIVATE pthread m)
        target_link_libraries(z_pub PRIVATE pthread m)
    endif()
endif()