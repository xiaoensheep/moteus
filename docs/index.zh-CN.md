# moteus 无刷伺服文档

欢迎来到 moteus 无刷电机控制器（brushless motor controller）的文档站点。

## 什么是 moteus？

moteus 控制器是高性能模块化无刷电机控制器，集成同轴磁编码器（magnetic encoder），专为机器人应用设计。其特点包括：

- **磁场定向控制（Field Oriented Control，FOC）**，用于三相无刷电机
- **集成磁编码器**，用于精确位置检测
- **高速 CAN-FD 通信**，速率为 5Mbps
- **多种控制模式**：位置、速度与转矩控制，支持加速度和速度受限的轨迹
- **快速控制环路**，运行于 15-30kHz

## 硬件型号

| 名称   | 输入电压 | 峰值功率     | 质量  | 尺寸        |
|--------|----------|--------------|-------|-------------|
| r4.11  | 10-44V   | 900W @ 30V   | 14.2g | 46x53mm     |
| c1     | 10-51V   | 250W @ 28V   | 8.9g  | 38x38x9mm   |
| n1     | 10-54V   | 2kW @ 36V    | 14.6g | 46x46x8mm   |
| x1     | 10-54V   | 1.3kW @ 36V  | 23.8g | 56x56x10mm  |

已组装并测试的电路板可在 [mjbots.com](https://mjbots.com) 购买。

## 文档

- [快速入门](quick-start.zh-CN.md)
- [使用 Moteus](guides/mechanical-setup.zh-CN.md)
- [集成](integration/python.zh-CN.md)
- [故障排查](troubleshooting/calibration.zh-CN.md)
- [参考](reference/pinouts.zh-CN.md)

## 社区与支持

- [Discord 社区](https://discord.gg/W4hUpBb)
- [GitHub 仓库](https://github.com/mjbots/moteus)
- [购买硬件](https://mjbots.com)

## 许可证

本仓库中的所有文件均在 [Apache 2.0 许可证](https://www.apache.org/licenses/LICENSE-2.0) 下提供。

!!! note "商标声明"
    mjbots Robotic Systems LLC 拥有并保护 "mjbots" 与 "moteus" 商标。如果您想在您的项目中使用这些名称，请阅读[商标政策](https://mjbots.com/trademark-policy)。
