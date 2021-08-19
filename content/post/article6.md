---
title: "Writing a simple Linux Kernel on Armv8 - 5"
date: 2021-05-26T00:22:56-04:00
author: "ozan ocak"
tags: [
    "aarch64",
    "kernel",
    "linux",
]
series : ["kernel development"]
---
<h5>Boot to Exception level 1</h5>


We need to add system register header file then activate neccessary registers in master section before return it , lastly we jump exception level 1 in order to allocate memory in bss section.

```boot.S
#include "sysregs.h"

master:
    ldr x0, =SCTLR_VALUE_MMU_DISABLED
    msr sctlr_el1, x0

    ldr x0, =HCR_VALUE
    msr hcr_el2, x0

    ldr x0, =SCR_VALUE
    msr scr_el3, x0

    ldr x0, =SPSR_VALUE
    msr spsr_el3, x0

    adr x0, el1_entry
    msr elr_el3, x0

    eret

el1_entry:
    adr x0, bss_begin
    adr x1, bss_end
    sub x1, x1, x0
    bl memzero

```

All the system register information base on the booklet of Arm architecture reference manual Armv8. by pushing binary numbers on the left or right side we can locate right registers.

```sysregs.h
#pragma once

//D13.2.113

#define SCTLR_RESERVED                  (3 << 28) | (3 << 22) | (1 << 20) | (1 << 11)
#define SCTLR_EE_LITTLE_ENDIAN          (0 << 25)
#define SCTLR_EOE_LITTLE_ENDIAN         (0 << 24)
#define SCTLR_I_CACHE_DISABLED          (0 << 12)
#define SCTLR_D_CACHE_DISABLED          (0 << 2)
#define SCTLR_MMU_DISABLED              (0 << 0)
#define SCTLR_MMU_ENABLED               (1 << 0)

#define SCTLR_VALUE_MMU_DISABLED (SCTLR_RESERVED | SCTLR_EE_LITTLE_ENDIAN | SCTLR_I_CACHE_DISABLED | SCTLR_D_CACHE_DISABLED | SCTLR_MMU_DISABLED)

//D13.2.47

#define HCR_RW                          (1 << 31)
#define HCR_VALUE                       HCR_RW

//D13.2.112

#define SCR_RESERVED                    (3 << 4)
#define SCR_RW                          (1 << 10)
#define SCR_NS                          (1 << 0)
#define SCR_VALUE                       (SCR_RESERVED | SCR_RW | SCR_NS)

//C5.2.19

#define SPSR_MASK_ALL                   (7 << 6)
#define SPSR_EL1h                       (5 << 0)
#define SPSR_EL2h                       (9 << 0)
#define SPSR_VALUE                      (SPSR_MASK_ALL | SPSR_EL1h)
```

