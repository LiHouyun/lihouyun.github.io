---
title: STM32 GPIO ：从 BSRR 寄存器到实际引脚状态
data: 2025-10-10 21:10:00 +0800
category: 软件
tag: [嵌入式, STM32, GPIO]
description: STM32 GPIO ：从 BSRR 寄存器到实际引脚状态。
image: ../assets/img-md/STM32GPIO：从BSRR寄存器到实际引脚状态/1-light.png
---

在 STM32 开发中，GPIO（General Purpose Input/Output，通用输入/输出口）引脚操作是最基础也最核心的功能之一。无论是驱动LED、读取传感器还是控制外设，都离不开对 GPIO 引脚的电平控制。本文将以一个具体的示例为切入点，深入解析 HAL 库函数操作背后的寄存器逻辑，以及引脚状态的实际变化规律。

## 1. 一个简单的 GPIO 操作示例

一段常见的 GPIO 操作代码：
```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_SET);    // 置高电平
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_RESET);  // 置低电平
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_SET);    // 再次置高
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_RESET);  // 再次置低
```

这段代码反复对 GPIOA 的第7个引脚（GPIO_PIN_7）进行高低电平切换。如果该引脚连接了一个LED（高电平点亮、低电平熄灭），最终LED会是什么状态？要弄清楚这个问题，我们需要先了解 HAL 库函数背后的寄存器操作。

## 2. HAL_GPIO_WritePin 函数的底层逻辑：BSRR 寄存器

STM32 的 GPIO 引脚电平控制主要通过 **BSRR 寄存器**（Bit Set/Reset Register，位设置/清除寄存器）实现。HAL 库的 `HAL_GPIO_WritePin` 函数本质上是对该寄存器的封装，理解 BSRR 寄存器的工作原理是掌握 GPIO 操作的关键。

### 2.1 BSRR 寄存器的结构与功能
BSRR 是一个 32 位的只写寄存器，分为两个部分：
- **低 16 位（bit 0~15）**：复位位（Reset）。当某一位置 1 时，对应引脚被设置为低电平。
- **高 16 位（bit 16~31）**：置位位（Set）。当某一位置 1 时，对应引脚被设置为高电平。

例如，对于 GPIOA_PIN_7：
- 若要置高电平（GPIO_PIN_SET），需将 BSRR 的 bit 23（16+7=23）置 1；
- 若要置低电平（GPIO_PIN_RESET），需将 BSRR 的 bit 7 置 1。


## 3. 分步解析代码执行过程

我们按顺序分析示例代码中 BSRR 寄存器的变化和引脚状态：

### 3.1 第一条指令：`HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_SET);`
- 操作：将 BSRR 的 bit 23 置 1（高 16 位第7位）。
- BSRR 状态：`0x00800000`（`0000 0000 1000 0000 0000 0000 0000 0000`，仅 bit 23 为 1）。
- 引脚状态：高电平 → LED 点亮。


### 3.2 第二条指令：`HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_RESET);`
- 操作：将 BSRR 的 bit 7 置 1（低 16 位第7位）。
- BSRR 状态：`0x00800080`（`0000 0000 1000 0000 0000 0000 1000 0000`，bit 7 和 bit 23 均为 1，寄存器写 1 后不会自动清零）。
- 引脚状态：低电平 → LED 熄灭。


### 3.3 第三条指令：`HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_SET);`
- 操作：再次将 BSRR 的 bit 23 置 1（已为 1，无实际变化）。
- BSRR 状态：保持 `0x00800080`。
- 引脚状态：高电平 → LED 重新点亮。


### 3.4 第四条指令：`HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_RESET);`
- 操作：再次将 BSRR 的 bit 7 置 1（已为 1，无实际变化）。
- BSRR 状态：保持 `0x00800080`。
- 引脚状态：低电平 → LED 再次熄灭。


## 4. 关键知识点：BSRR 操作的优先级与独立性

当 BSRR 的置位位和复位位同时为 1 时（如示例中后两条指令执行后），引脚状态为何由最后一次操作决定？

这里需要明确 BSRR 寄存器的两个重要特性：

1. **单次操作的优先级**：  
   若**同一时刻**对同一引脚的置位位（高16位）和复位位（低16位）同时写 1（例如直接执行 `GPIOA->BSRR = 0x00800080;`），则**置位操作优先级更高**，最终引脚为高电平。这是硬件设计确保的逻辑，避免了冲突。

2. **多次操作的独立性**：  
   示例中的四条指令是**按顺序独立执行**的，每次仅操作置位位或复位位中的一个，不存在“同时写 1”的情况。因此，每次操作都会覆盖上一次的引脚状态，最终状态由**最后一次操作**决定。

## 5. 最终结论

示例代码执行完毕后：
- BSRR 寄存器状态为 `0x00800080`（bit 7 和 bit 23 均为 1）；
- LED 状态为**熄灭**（最后一次操作为复位，引脚为低电平）。
