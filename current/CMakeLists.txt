cmake_minimum_required(VERSION 3.18)
project(urg_library LANGUAGES C)

# Define base URG sources (cross-platform)
set(URG_BASE_SOURCES
    src/urg_connection.c
    src/urg_debug.c
    src/urg_ring_buffer.c
    src/urg_sensor.c
    src/urg_serial.c
    src/urg_serial_utils.c
    src/urg_tcpclient.c
    src/urg_utils.c
)

# Add platform-specific sources
if(WIN32)
    set(URG_PLATFORM_SOURCES
        src/urg_serial_windows.c
        src/urg_serial_utils_windows.c
    )
else()
    # Both Linux and macOS use the same Linux implementation
    # The OS detection is handled in the source code via urg_detect_os.h
    # Note: urg_serial_linux.c is included by urg_serial.c, not compiled separately
    set(URG_PLATFORM_SOURCES
        src/urg_serial_utils_linux.c
    )
endif()

# Combine all sources
set(URG_SOURCES ${URG_BASE_SOURCES} ${URG_PLATFORM_SOURCES})

add_library(urg_c STATIC ${URG_SOURCES})
target_include_directories(urg_c PUBLIC include/c)

# Platform-specific configuration
if(WIN32)
    # Windows-specific settings
    target_compile_definitions(urg_c PRIVATE
        WIN32
        _CONSOLE
        URG_WINDOWS_OS
    )
    target_compile_options(urg_c PRIVATE /wd4996)
    target_link_libraries(urg_c PUBLIC wsock32 setupapi)
    
elseif(APPLE)
    # macOS-specific settings
    target_compile_definitions(urg_c PRIVATE
        URG_MAC_OS
        URG_LINUX_OS  # macOS uses Linux implementation
    )
    # Ensure O_EXLOCK compatibility (macOS has O_EXLOCK natively)
    # Add any macOS-specific system libraries if needed
    
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    # Linux-specific settings
    target_compile_definitions(urg_c PRIVATE
        URG_LINUX_OS
        O_EXLOCK=0  # Linux doesn't have O_EXLOCK, define as 0
    )
    
else()
    # Other Unix-like systems (fallback to Mac behavior as per original code)
    target_compile_definitions(urg_c PRIVATE
        URG_MAC_OS
        URG_LINUX_OS
        O_EXLOCK=0  # Define O_EXLOCK for compatibility
    )
endif()

# Add required system libraries for Unix-like systems
if(UNIX)
    # No additional libraries needed for basic POSIX serial communication
    # Libraries like pthread, rt, etc. are typically linked automatically
endif()

# Ensure proper compiler flags for all platforms
if(NOT WIN32)
    # Add POSIX feature test macros to ensure all required functions are available
    target_compile_definitions(urg_c PRIVATE
        _POSIX_C_SOURCE=200809L  # Ensure POSIX.1-2008 features
        _DEFAULT_SOURCE          # Ensure glibc extensions (Linux)
        _BSD_SOURCE             # Ensure BSD extensions
        _GNU_SOURCE             # Enable GNU extensions for Linux
    )
endif()

# Handle platform-specific missing definitions and includes
# These definitions solve the compilation issues without modifying source files
if(APPLE)
    # macOS specific: Handle missing baud rates that might not be defined
    target_compile_definitions(urg_c PRIVATE
        # Ensure all baud rates are available
        _DARWIN_C_SOURCE
    )
    # macOS has O_EXLOCK natively, no need to define it
    
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    # Linux specific: Define missing constants
    target_compile_definitions(urg_c PRIVATE
        # Handle missing baud rates
        B57600=0010001
        B115200=0010002
    )
    # O_EXLOCK already defined as 0 above
    
else()
    # Other Unix systems: provide fallback definitions
    target_compile_definitions(urg_c PRIVATE
        # Handle missing baud rates
        B57600=0010001
        B115200=0010002
    )
endif()