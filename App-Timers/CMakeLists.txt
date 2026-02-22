cmake_minimum_required(VERSION 3.14)

# Make project data accessible to compiler
add_compile_definitions(APP_NAME="${APP_4_NAME}")
add_compile_definitions(APP_VERSION="${APP_4_VERSION_NUMBER}")
add_compile_definitions(BUILD_NUM=${BUILD_NUMBER})

# Include app source code file(s)
add_executable(${APP_4_NAME}
    ${APP_4_SRC_DIRECTORY}/main.cpp
    ${COMMON_CODE_DIRECTORY}/utils.cpp
)

# Link to built libraries
target_link_libraries(${APP_4_NAME} LINK_PUBLIC
    pico_stdlib
    FreeRTOS)
if (PICO_CYW43_SUPPORTED)
    target_link_libraries(${APP_4_NAME} LINK_PUBLIC
        pico_cyw43_arch_none)
endif()

# Enable extra build products
pico_add_extra_outputs(${APP_4_NAME})

# Modify the below lines to enable/disable output over UART/USB
pico_enable_stdio_uart(${APP_4_NAME} 0)
pico_enable_stdio_usb(${APP_4_NAME} 1)
