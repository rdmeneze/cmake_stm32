cmake_minimum_required( VERSION 3.15.3 )

set( CMAKE_VERBOSE_MAKEFILE ON )

set( CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/arm-none-eabi-gcc.cmake )


project( l432kc )
set( EXECUTABLE ${PROJECT_NAME}.elf )
set( LINKER_FILE ${CMAKE_SOURCE_DIR}/BSP/STM32L432KCUx_FLASH.ld )

enable_language( C ASM )
set( CMAKE_C_STANDARD 99 )
set( CMAKE_C_STANDARD_REQUIRED ON )
set( CMAKE_C_EXTENSIONS OFF )

message(STATUS "CMAKE_TOOLCHAIN_FILE is: ${CMAKE_TOOLCHAIN_FILE}")

set(STM32CUBEMX_GENERATED_FILES 
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_tim.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_tim_ex.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_uart.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_uart_ex.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_i2c.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_i2c_ex.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_rcc.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_rcc_ex.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_flash.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_flash_ex.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_flash_ramfunc.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_gpio.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_dma.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_dma_ex.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_pwr.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_pwr_ex.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_cortex.c
	BSP/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_exti.c

	BSP/Drivers/CMSIS/Device/ST/STM32L4xx/Source/Templates/system_stm32l4xx.c
	# BSP/Drivers/CMSIS/Device/ST/STM32L4xx/Source/Templates/gcc/startup_stm32l432xx.s

	# BSP/Inc/main.h
	# BSP/Inc/stm32f4xx_hal_conf.h
	# BSP/Inc/stm32f4xx_it.h

	BSP/Src/main.c
	BSP/Src/stm32l4xx_it.c
	BSP/Src/stm32l4xx_hal_msp.c

	BSP/startup_stm32l432xx.s
	)


add_executable( ${EXECUTABLE} ${STM32CUBEMX_GENERATED_FILES} )

target_compile_definitions( ${EXECUTABLE} PRIVATE 
	-DUSE_HAL_DRIVER
	-DSTM32L432xx
	)

target_include_directories( ${EXECUTABLE} PRIVATE 
	BSP/Inc
	BSP/Drivers/STM32L4xx_HAL_Driver/Inc
	BSP/Drivers/CMSIS/Device/ST/STM32L4xx/Include
	BSP/Drivers/CMSIS/Include
	)


target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=cortex-m4
        -mthumb
        -mfpu=fpv4-sp-d16
        -mfloat-abi=hard

        -fdata-sections
        -ffunction-sections

        -Wall

        $<$<CONFIG:Debug>:-Og>
        )

target_link_options( ${EXECUTABLE} PRIVATE
	-T${LINKER_FILE}
	-mcpu=cortex-m4
	-mthumb
	-mfpu=fpv4-sp-d16
	-mfloat-abi=hard
	-specs=nano.specs
	-lc
	-lm
	-lnosys
	-Wl,-Map=${PROJECT_NAME}.map,--cref
	-Wl,--gc-sections
	)



# Print executable size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-size ${EXECUTABLE})

# Create hex file
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-objcopy -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND arm-none-eabi-objcopy -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)


