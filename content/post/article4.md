---
title: "Writing a simple Linux Kernel on Armv8 - 3 "
date: 2021-05-24T00:10:56-04:00
author: "ozan ocak"
tags: [
    "aarch64",
    "kernel",
    "linux",
]
---

<h5>Kernel boot process preparation</h5>

Now we can start adding header files under include folder. We will first create aux.h base.h and gpio.h files under peripherals subfolder. Raspberry Pi3 starts from 0x3F000000 adress, below is reserved for interrupt vector table and Raspberry Pi starts from 0xFE000000 adress.

```base.h
#pragma once

#if RPI_VERSION == 3
#define PBASE 0x3F000000  

#elif RPI_VERSION == 4
#define PBASE 0xFE000000

#else
#define PBASE 0
#error RPI_VERSION NOT DEFINED

#endif
```
We can rename data types shorter names for convinient. volatile type uses only for registers of cpu thus we rename it reg32 in order to remember we process data on actual register.

```common.h
#pragma once

#include <stdint.h>

typedef uint8_t u8;
typedef uint16_t u16;
typedef uint32_t u32;
typedef uint64_t u64;

typedef volatile u32 reg32;

```
Then header file for memory management which is mm.h . This file fedine page size and low memory. 

```mm.h
#pragma once

#define PAGE_SHIFT 12
#define TABLE_SHIFT 9
#define SECTION_SHIFT (PAGE_SHIFT + TABLE_SHIFT)
#define PAGE_SIZE (1 << PAGE_SHIFT)
#define SECTION_SIZE (1 << SECTION_SHIFT)

#define LOW_MEMORY (2 * SECTION_SIZE)

#ifndef __ASSEMBLER__

void memzero(unsigned long src, unsigned int n);

#endif

```
Then we need define some utility functions for now.

```utils.h
#pragma once

#include "common.h"

void delay(u64 ticks);
void put32(u64 address, u32 value);
u32 get32(u64 address);

```

After defining header file we can write the implementation of the memzero function.

```mm.S
.globl memzero
memzero:
    str xzr, [x0], #8
    subs x1, x1, #8
    b.gt memzero
    ret
```
We take the Axilery information from Broadcast Arm cpu booklet in auxilery chapters

```aux.h
#pragma once

#include "common.h"

#include "peripherals/base.h"

struct AuxRegs {
    reg32 irq_status;
    reg32 enables;
    reg32 reserved[14];
    reg32 mu_io;
    reg32 mu_ier;
    reg32 mu_iir;
    reg32 mu_lcr;
    reg32 mu_mcr;
    reg32 mu_lsr;
    reg32 mu_msr;
    reg32 mu_scratch;
    reg32 mu_control;
    reg32 mu_status;
    reg32 mu_baud_rate;
};

#define REGS_AUX ((struct AuxRegs *)(PBASE + 0x00215000))
```

Now we can initialize our tiny kernel from kernel.c

```kernel.c
#include "common.h"
#include "mini_uart.h"

void kernel_main() {
    uart_init();
    uart_send_string("Rasperry PI Bare Metal OS Initializing...\n");

#if RPI_VERSION == 3
    uart_send_string("\tBoard: Raspberry PI 3\n");
#endif

#if RPI_VERSION == 4
    uart_send_string("\tBoard: Raspberry PI 4\n");
#endif

    uart_send_string("\n\nDone\n");

    while(1) {
        uart_send(uart_recv());
    }
}
```