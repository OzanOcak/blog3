---
title: "Writing a simple Linux Kernel on Armv8 -4 "
date: 2021-05-26T00:10:56-04:00
author: "ozan ocak"
tags: [
    "aarch64",
    "kernel",
    "linux",
]
series : ["kernel development"]
---
<h5>GPIO of Arm</h5>


We take the GPIO information from Broadcast Arm cpu booklet in gpio chapters

```gpio.h

#pragma once

#include "peripherals/gpio.h"

typedef enum _GpioFunc {
    GFInput = 0,
    GFOutput = 1,
    GFAlt0 = 4,
    GFAlt1 = 5,
    GFAlt2 = 6,
    GFAlt3 = 7,
    GFAlt4 = 3,
    GFAlt5 = 2
} GpioFunc;

void gpio_pin_set_func(u8 pinNumber, GpioFunc func);

void gpio_pin_enable(u8 pinNumber);

```

Now we can write gpio.c file. we first locate bits with and gate then set them with or gate.

```gpio.c
#include "gpio.h"
#include "utils.h"

void gpio_pin_set_func(u8 pinNumber, GpioFunc func) {
    u8 bitStart = (pinNumber * 3) % 30;
    u8 reg = pinNumber / 10;

    u32 selector = REGS_GPIO->func_select[reg];
    selector &= ~(7 << bitStart);
    selector |= (func << bitStart);

    REGS_GPIO->func_select[reg] = selector;
}

void gpio_pin_enable(u8 pinNumber) {
    REGS_GPIO->pupd_enable = 0;
    delay(150);
    REGS_GPIO->pupd_enable_clocks[pinNumber / 32] = 1 << (pinNumber % 32);
    delay(150);
    REGS_GPIO->pupd_enable = 0;
    REGS_GPIO->pupd_enable_clocks[pinNumber / 32] = 0;
}
```

<h5>Mini uart connection</h5>
First define all the function to send the data

```mini_uart.h
#pragma once

void uart_init();
char uart_recv();
void uart_send(char c);
void uart_send_string(char *str);

```
Then the implementation of the functions in order to enable gpio pins and send string via uart

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