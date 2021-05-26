---
title: "Writing a simple Linux Kernel on Armv8 - 2"
date: 2021-05-22T00:10:56-04:00
author: "ozan ocak"
tags: [
    "aarch64",
    "kernel",
    "linux",
]
---

This is a documentation of writing bare operating system on hardware specifically on raspberry pi. This doc originally developed by Sergey Matyukevich then created another tutorial upon original project by Rocky Pulley.

<h3>Makefile</h3> 
We first need to specify the environment for booting in config.txt and create a makefile in order to not to call all dependencies every single time when we build the project. In the makefile we locate boot directory then determine the type of arm gnu

``` Makefile
RPI_VERSION ?= 4

BOOTMNT ?= /media/parallels/boot

ARMGNU ?= aarch64-linux-gnu

COPS = -DRPI_VERSION=$(RPI_VERSION) -Wall -nostdlib -nostartfiles -ffreestanding \
	   -Iinclude -mgeneral-regs-only

ASMOPS = -Iinclude

BUILD_DIR = build
SRC_DIR = src

all : kernel8.img

clean :
	rm -rf $(BUILD_DIR) *.img 

$(BUILD_DIR)/%_c.o: $(SRC_DIR)/%.c
	mkdir -p $(@D)
	$(ARMGNU)-gcc $(COPS) -MMD -c $< -o $@

$(BUILD_DIR)/%_s.o: $(SRC_DIR)/%.S
	mkdir -p $(@D)
	$(ARMGNU)-gcc $(COPS) -MMD -c $< -o $@

C_FILES = $(wildcard $(SRC_DIR)/*.c)
ASM_FILES = $(wildcard $(SRC_DIR)/*.S)
OBJ_FILES = $(C_FILES:$(SRC_DIR)/%.c=$(BUILD_DIR)/%_c.o)
OBJ_FILES += $(ASM_FILES:$(SRC_DIR)/%.S=$(BUILD_DIR)/%_s.o)

DEP_FILES = $(OBJ_FILES:%.o=%.d)
-include $(DEP_FILES)

kernel8.img: $(SRC_DIR)/linker.ld $(OBJ_FILES)
	@echo "Building for RPI $(value RPI_VERSION)"
	@echo "Deploy to $(value BOOTMNT)"
	@echo ""
	$(ARMGNU)-ld -T $(SRC_DIR)/linker.ld -o $(BUILD_DIR)/kernel8.elf $(OBJ_FILES)
	$(ARMGNU)-objcopy $(BUILD_DIR)/kernel8.elf -O binary kernel8.img
ifeq ($(RPI_VERSION), 4)
	cp kernel8.img $(BOOTMNT)/kernel8-rpi4.img
else
	cp kernel8.img $(BOOTMNT)/
endif
	cp config.txt $(BOOTMNT)/
	sync
```

###### we need to add all necessary flags
* Wall for all warnings
* nostdlib for not  calling  C standard library
* ffreestanding means compiler not to assume standard functions have regular defination
* linclude is header files in the include folder
* mgeneral-regs-only is to use only general purpose register of arm processor

While SRC_DIR contain source code, BUILD_DIR contains compiled object files. Build target will be image file( kernel8.img )

<h5>The linker script</h5>
purpose of the linker script is to describe how the sections in the input object files should be mapped into the output file (.elf).

```linker.ld
SECTIONS
{
    .text.boot : { *(.text.boot) }
    .text : { *(.text) }
    .rodata : { *(.rodata) }
    .data : { *(.data) }
    . = ALIGN(0x8);
    bss_begin = .;
    .bss : { *(.bss*) }
    bss_end = .;
}
```
After startup, the Raspberry Pi loads(kernel8.img)into memory and starts execution from the beginning of the file. That's why the(.text.boot)section must be first; we are going to put the OS startup code inside this section. The.text,.rodata, and.datasections contain kernel-compiled instructions, read-only data, and normal data

<h5>Booting the kernel</h5>
Now it is time to boot by boot.S file where we check process id (and with hex 1111)  then hang all processes . After cleaning the .bss section, we initialize the stack pointer and pass execution to the kernel_main function. 

```boot.S
#include "mm.h"

.section ".text.boot"

.globl _start
_start:
    mrs x0, mpidr_el1
    and x0, x0, #0xFF
    cbz x0, master
    b proc_hang

master:
    adr x0, bss_begin
    adr x1, bss_end
    sub x1, x1, x0
    bl memzero

    mov sp, #LOW_MEMORY
    bl kernel_main
    b  proc_hang

proc_hang:
    wfe
    b proc_hang
```

LOW_MEMORY is defined in mm.h and is equal to 4MB.
The kernel_main function
We have seen that the boot code eventually passes control to the kernel_main function.

<h5>Mini UART initialization</h5>

Put32 and get32  functions are very simple; they allow us to read and write some data to and from a 32-bit register in utils.S. After the Mini UART is ready, we can try to use it to send and receive some data

```mini_uart.c
#include "gpio.h"
#include "utils.h"
#include "peripherals/aux.h"
#include "mini_uart.h"

#define TXD 14
#define RXD 15

void uart_init() {
    gpio_pin_set_func(TXD, GFAlt5);
    gpio_pin_set_func(RXD, GFAlt5);

    gpio_pin_enable(TXD);
    gpio_pin_enable(RXD);

    REGS_AUX->enables = 1;
    REGS_AUX->mu_control = 0;
    REGS_AUX->mu_ier = 0;
    REGS_AUX->mu_lcr = 3;
    REGS_AUX->mu_mcr = 0;

#if RPI_VERSION == 3
    REGS_AUX->mu_baud_rate = 270; // = 115200 @ 250 Mhz
#endif

#if RPI_VERSION == 4
    REGS_AUX->mu_baud_rate = 541; // = 115200 @ 500 Mhz
#endif

    REGS_AUX->mu_control = 3;

    uart_send('\r');
    uart_send('\n');
    uart_send('\n');
}

void uart_send(char c) {
    while(!(REGS_AUX->mu_lsr & 0x20));

    REGS_AUX->mu_io = c;
}

char uart_recv() {
    while(!(REGS_AUX->mu_lsr & 1));

    return REGS_AUX->mu_io & 0xFF;
}

void uart_send_string(char *str) {
    while(*str) {
        if (*str == '\n') {
            uart_send('\r');
        }

        uart_send(*str);
        str++;
    }
}
```













