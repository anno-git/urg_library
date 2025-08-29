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
    set(URG_PLATFORM_SOURCES
        src/urg_serial_linux.c
        src/urg_serial_utils_linux.c
    )
endif()

# Combine all sources
set(URG_SOURCES ${URG_BASE_SOURCES} ${URG_PLATFORM_SOURCES})

add_library(urg_c STATIC ${URG_SOURCES})
target_include_directories(urg_c PUBLIC include/c)

if(WIN32)
    target_compile_definitions(urg_c PRIVATE WIN32 _CONSOLE)
    target_compile_options(urg_c PRIVATE /wd4996)
    target_link_libraries(urg_c PUBLIC wsock32 setupapi)
endif()