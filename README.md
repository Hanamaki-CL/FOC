# STM32 电机矢量控制实验工程

基于 STM32G431 和 STM32 HAL 的 PMSM / BLDC 电机控制工程，围绕三相驱动、电流采样、磁编码器反馈和 FOC 算法组织代码，供电机控制学习、移植和调试使用。

本文依据当前仓库代码介绍工程结构、功能模块和运行状态。

## 工程状态

当前 `Core/Src/main.c` 初始化外设后，进入 MT6816 编码器读取与 USB CDC 输出循环。工程包含电机状态机、ADC 回调、CAN 命令处理和 FOC 模块，但当前主程序尚未调用 `RobotInit()` 和 `RobotTask()`。运行完整电机控制需要先检查初始化、回调连接和硬件适配。

`App/README.md` 和其他模块文档提供原工程设计说明，部分内容与当前入口或代码可能存在差异。请结合源码核对。

## 控制链路

工程中的 FOC 模块包含以下计算链路，ADC 注入转换完成回调位于 `App/motor_task.c`。

```text
相电流采样 + 编码器角度
          ↓
    Clarke / Park 变换
          ↓
      Id / Iq 调节
          ↓
     逆 Park / SVPWM
          ↓
     TIM1 三相 PWM 输出
```

代码还包含电流、速度和位置控制模式、斜坡与轨迹处理、状态机、FDCAN 通信和故障处理逻辑。各模块的接入情况与运行效果需要在目标硬件上验证。

## 硬件配置

| 项目 | 工程配置 |
| --- | --- |
| 微控制器 | STM32G431CBT6 |
| 三相驱动 | TIM1 互补 PWM，FD6288 栅极驱动 |
| 电流反馈 | ADC 注入组采样，三电阻采样方案 |
| 位置反馈 | MT6816 磁编码器，SPI 接口 |
| 通信接口 | FDCAN、USART1、USB CDC |

引脚和外设的具体配置可在 `milFOC.ioc` 和 `Core/Src/` 中查看，移植时应与实际控制板的原理图核对。

## 代码结构

| 目录 | 内容 |
| --- | --- |
| `Core/` | STM32CubeMX 外设初始化和主程序入口 |
| `BSP/` | ADC、CAN、串口等板级驱动 |
| `Modules/motor/` | FOC 变换、调制和电机控制逻辑 |
| `Modules/encoder/` | MT6816 角度读取及速度估算 |
| `Modules/controller/` | PID 控制器 |
| `Modules/comm/` | CAN 数据封装和通信 |
| `App/` | 应用初始化、命令处理和 ADC 回调 |
| `Drivers/`、`Middlewares/` | HAL、CMSIS 和第三方组件 |

## 构建

工程提供 CMake 和 Ninja 配置，需要安装 CMake、Ninja 及 Arm GNU 工具链，并将 `arm-none-eabi-gcc` 加入 PATH。

```bash
cmake --preset Debug
cmake --build --preset Debug
```

配置文件使用 `cmake/gcc-arm-none-eabi.cmake`，生成目录为 `build/Debug/`。也可使用 STM32CubeMX 打开 `milFOC.ioc` 查看外设配置。

## 许可

代码包含第三方组件及版权声明，各组件的许可条款见对应文件和目录。当前仓库根目录未包含项目级许可证文件。
