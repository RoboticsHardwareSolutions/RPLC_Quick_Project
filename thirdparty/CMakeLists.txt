####################### FREERTOS lib ############################
add_library(freertos_config INTERFACE)

target_include_directories(freertos_config SYSTEM
        INTERFACE
        ../Core/Inc
)

target_compile_definitions(freertos_config
        INTERFACE
        projCOVERAGE_TEST=0
)

set(FREERTOS_HEAP "4" CACHE STRING "" FORCE)

get_directory_property(COMPILE_OPTS COMPILE_OPTIONS)
if(COMPILE_OPTS MATCHES "-mcpu=([^ ;]+)")
        set(TARGET_CPU ${CMAKE_MATCH_1})
        if(TARGET_CPU STREQUAL "cortex-m3")
                set(FREERTOS_PORT "GCC_ARM_CM3" CACHE STRING "" FORCE)
        elseif(TARGET_CPU STREQUAL "cortex-m4")
                if(COMPILE_OPTS MATCHES "-mfloat-abi=([^ ;]+)")
                        set(TARGET_FPU ${CMAKE_MATCH_1})
                        if(TARGET_FPU STREQUAL "hard")
                                set(FREERTOS_PORT "GCC_ARM_CM4F" CACHE STRING "" FORCE)
                        else()
                                set(FREERTOS_PORT "GCC_ARM_CM4_MPU" CACHE STRING "" FORCE)
                        endif()
                else()
                        set(FREERTOS_PORT "GCC_ARM_CM4_MPU" CACHE STRING "" FORCE)
                endif()
        elseif(TARGET_CPU STREQUAL "cortex-m7")
                set(FREERTOS_PORT "GCC_ARM_CM7" CACHE STRING "" FORCE)
        endif()
endif()

add_subdirectory(freertos)

####################### RTT lib ############################
add_library(RTT STATIC
        RTT/RTT/SEGGER_RTT_ASM_ARMv7M.S
        RTT/RTT/SEGGER_RTT_printf.c
        RTT/RTT/SEGGER_RTT.c
)

target_compile_definitions(RTT PUBLIC -DBUFFER_SIZE_DOWN=64)

target_include_directories(RTT
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/RTT/RTT>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/RTT/Config>
)

####################### USB lib ############################
if(RPLC_M)
        add_library(usb_core STATIC libusb_stm32/src/usbd_core.c libusb_stm32/src/usbd_stm32f103_devfs.c)
        target_compile_definitions(usb_core PUBLIC -DUSB_PMASIZE=512)
        target_include_directories(usb_core
                PUBLIC
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/libusb_stm32/inc>
        )
endif()

####################### LWIP lib ############################
if(RPLC_XL)
        set(LWIP_DIR ${CMAKE_CURRENT_SOURCE_DIR}/lwip)
        include(${LWIP_DIR}/src/Filelists.cmake)

        add_library(lwip ${lwipnoapps_SRCS})
        target_include_directories(lwip PUBLIC ${LWIP_DIR}/src/include)
        target_link_libraries(lwip PUBLIC freertos_kernel lwip_service)
endif()

####################### RHS core lib ############################
add_subdirectory(rhs)
