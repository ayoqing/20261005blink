# 薛俪卿 20263001793 第一次作业 10.24

STM32F103C8T6 三色 LED 流水灯

---

## 一、代码

主循环逻辑（`Core/Src/main.c`）：

```c
  MX_GPIO_Init();
  int state=1;

  while (1)
  {
    if (state<=2||state==5) {
      HAL_GPIO_WritePin(LED_red_GPIO_Port, LED_red_Pin, GPIO_PIN_SET);
      HAL_Delay(300);
    }else {
      HAL_GPIO_WritePin(LED_red_GPIO_Port, LED_red_Pin, GPIO_PIN_RESET);
      HAL_Delay(300);
    }

    if (state>=1&&state<=3) {
      HAL_GPIO_WritePin(LED_blue_GPIO_Port, LED_blue_Pin, GPIO_PIN_SET);
      HAL_Delay(300);
    }else {
      HAL_GPIO_WritePin(LED_blue_GPIO_Port, LED_blue_Pin, GPIO_PIN_RESET);
      HAL_Delay(300);
    }

    if (state>=2&&state<=4) {
      HAL_GPIO_WritePin(LED_green_GPIO_Port, LED_green_Pin, GPIO_PIN_SET);
      HAL_Delay(300);
    } else {
      HAL_GPIO_WritePin(LED_green_GPIO_Port, LED_green_Pin, GPIO_PIN_RESET);
      HAL_Delay(300);
    }

    HAL_Delay(200);
    state++;
    if (state>5) {
      state=0;
    }
  }
```

GPIO 初始化（`Core/Src/gpio.c`）：

```c
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* 蓝、绿 LED：PA6 / PA7 */
  GPIO_InitStruct.Pin   = LED_blue_Pin|LED_green_Pin;
  GPIO_InitStruct.Mode  = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull  = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  /* 红 LED：PB0 */
  GPIO_InitStruct.Pin = LED_red_Pin;
  HAL_GPIO_Init(LED_red_GPIO_Port, &GPIO_InitStruct);
```

---

## 二、亮灭情况

`state` 在 0 ~ 5 之间循环，每个状态停留约 1.1 秒（300ms × 3 + 200ms）：

| state | 红 (PB0) | 蓝 (PA6) | 绿 (PA7) |
|:-----:|:--------:|:--------:|:--------:|
| 0 | 亮 | 灭 | 灭 |
| 1 | 亮 | 亮 | 灭 |
| 2 | 亮 | 亮 | 亮 |
| 3 | 灭 | 亮 | 亮 |
| 4 | 灭 | 灭 | 亮 |
| 5 | 亮 | 灭 | 灭 |

实际效果：红 → 红蓝 → 红蓝绿 → 蓝绿 → 绿 → 红，形成三色流动。

> 程序从 `state = 1` 开始运行，一轮完整循环约 6.6 秒。LED 为高电平点亮。

---

## 三、开发环境

| 项目 | 说明 |
|------|------|
| MCU | STM32F103C8T6（Cortex-M3，LQFP48，64KB Flash / 20KB RAM） |
| 开发板 | Blue Pill 最小系统板 |
| 配置工具 | STM32CubeMX 6.18.1（Toolchain 选 CMake） |
| 库 | STM32F1xx HAL 库 |
| 编译器 | arm-none-eabi-gcc |
| 构建 | CMake + Ninja |
| IDE | CLion 2026.2 |
| 下载调试 | OpenOCD 0.12 + ST-Link V2（SWD 接口） |
| 系统时钟 | HSI 8 MHz，未启用 PLL |

---

## 四、引脚分配

| 引脚 | 标签 | 功能 | 配置 |
|------|------|------|------|
| PA6 | LED_blue | 蓝色 LED | 推挽输出，无上下拉，低速 |
| PA7 | LED_green | 绿色 LED | 推挽输出，无上下拉，低速 |
| PB0 | LED_red | 红色 LED | 推挽输出，无上下拉，低速 |
| PA13 | SWDIO | 调试数据线 | Serial Wire |
| PA14 | SWCLK | 调试时钟线 | Serial Wire |
| PD0 / PD1 | OSC_IN / OSC_OUT | 外部晶振（未使用） | — |
