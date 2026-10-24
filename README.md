# 20261005blink — STM32F103 三色 LED 流水灯

基于 STM32F103C8T6（Blue Pill 最小系统板）的三色 LED 呼吸式流水灯示例工程。
使用 STM32CubeMX 生成底层初始化代码，STM32 HAL 库驱动，CMake + `arm-none-eabi-gcc` 构建，
可在 CLion 中直接编译、下载与调试。

---

## 1. 功能简介

程序在 `main()` 的主循环中维护一个 `state` 变量，取值范围 `0 ~ 5`，循环递增。
每一轮循环根据 `state` 的当前值决定三颗 LED 的亮灭组合，配合 `HAL_Delay()`
形成"颜色渐变流动"的视觉效果。

### LED 亮灭状态表

| state | 红 (PB0) | 蓝 (PA6) | 绿 (PA7) |
|:-----:|:--------:|:--------:|:--------:|
| 0 | 亮 | 灭 | 灭 |
| 1 | 亮 | 亮 | 灭 |
| 2 | 亮 | 亮 | 亮 |
| 3 | 灭 | 亮 | 亮 |
| 4 | 灭 | 灭 | 亮 |
| 5 | 亮 | 灭 | 灭 |

> 状态 0 与状态 5 的显示效果相同，用于让"红→绿→红"的过渡自然衔接。

单个状态大约持续 `300ms × 3 + 200ms ≈ 1.1s`，一轮完整循环约 6.6 秒。

---

## 2. 硬件平台

| 项目 | 参数 |
|------|------|
| MCU | STM32F103C8T6（Cortex-M3，LQFP48） |
| Flash / RAM | 64 KB / 20 KB |
| 开发板 | Blue Pill 最小系统板 |
| 系统时钟 | HSI 8 MHz（未启用 PLL，SYSCLK = 8 MHz） |
| 调试接口 | SWD（PA13 = SWDIO，PA14 = SWCLK） |
| 调试器 | ST-Link V2 |

### 引脚分配

| 引脚 | 标签 | 功能 | 配置 |
|------|------|------|------|
| PA6 | `LED_blue` | 蓝色 LED | 推挽输出，无上下拉，低速 |
| PA7 | `LED_green` | 绿色 LED | 推挽输出，无上下拉，低速 |
| PB0 | `LED_red` | 红色 LED | 推挽输出，无上下拉，低速 |
| PD0 / PD1 | OSC_IN / OSC_OUT | 外部晶振（未使用） | — |

LED 采用**高电平点亮**（`GPIO_PIN_SET` 为亮）。若实际硬件为低电平驱动，
需在代码中把 `SET` / `RESET` 对调，或改为灌电流接法。

---

## 3. 目录结构

```
20261005blink/
├── Core/
│   ├── Inc/                     # 头文件（main.h、gpio.h、HAL 配置等）
│   ├── Src/                     # 源文件（main.c、gpio.c、中断与系统初始化）
│   └── Startup/                 # 启动文件 startup_stm32f103c8tx.s
├── Drivers/
│   ├── CMSIS/                   # Cortex-M3 内核与设备头文件
│   └── STM32F1xx_HAL_Driver/    # ST HAL 库
├── 20261005blink.ioc            # STM32CubeMX 工程文件（可直接打开重新生成）
├── CMakeLists.txt               # CMake 构建脚本
├── CMakeLists_template.txt      # CubeMX 生成 CMakeLists 的模板
├── STM32F103C8TX_FLASH.ld       # 链接脚本
└── README.md
```

> `cmake-build-debug/`、`.idea/`、`.settings/` 属于本地构建与 IDE 配置产物，已在 `.gitignore` 中排除。

---

## 4. 开发环境

| 工具 | 版本 / 说明 |
|------|-------------|
| STM32CubeMX | 6.18.1（Toolchain 选择 `CMake`） |
| 交叉编译器 | `arm-none-eabi-gcc` |
| 构建系统 | CMake + Ninja（或 MinGW Makefiles） |
| IDE | CLion 2026.2（可选，也可纯命令行构建） |
| 下载调试 | OpenOCD 0.12 + ST-Link V2 |

---

## 5. 编译

### 命令行构建

```bash
# 确保交叉编译器在 PATH 中（示例路径按实际安装位置调整）
export PATH="/d/arm.gcc/arm/bin:$PATH"

cmake -B build -G Ninja \
      -DCMAKE_BUILD_TYPE=Debug \
      -DCMAKE_C_COMPILER=arm-none-eabi-gcc \
      -DCMAKE_CXX_COMPILER=arm-none-eabi-g++ \
      -DCMAKE_ASM_COMPILER=arm-none-eabi-gcc

cmake --build build
```

构建产物为 `build/20261005blink.elf`（以及 `.hex` / `.bin`，取决于 CMake 配置）。

### CLion

直接用 CLion 打开工程根目录，等待 CMake 索引完成后点击构建即可。
注意本工程路径需为纯 ASCII 路径，`mingw32-make` 不支持中文路径。

---

## 6. 烧录

### 使用 OpenOCD + ST-Link

```bash
openocd -f interface/stlink.cfg \
        -f target/stm32f1x.cfg \
        -c "program build/20261005blink.elf verify reset exit"
```

### 使用 STM32CubeProgrammer

选择生成的 `.hex` 或 `.elf` 文件，接口选 SWD，即可一键下载。

---

## 7. 说明

- 本工程由 STM32CubeMX 生成，业务代码全部写在 `USER CODE BEGIN / END` 区块内，
  重新生成代码时不会被覆盖。
- 本地开发时可能存在 `Core/Inc/Backup`、`Core/Src/Backup` 等手工备份目录，
  已通过 `.gitignore` 的 `*.bak` 规则排除，不纳入版本管理。
- `system_stm32f1xx.c`、`stm32f1xx_hal_msp.c` 等为 HAL 库配套文件，请勿随意修改。

---

## 8. License

本工程中的 STM32 HAL 驱动与 CMSIS 部分版权归 STMicroelectronics 所有，
遵循其原始许可条款；应用层代码可自由使用与修改。
