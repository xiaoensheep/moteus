# 无运动故障排查

你向 moteus 发送了命令，或运行了脚本，但没有任何东西运动。本文将描述你可以采取哪些步骤来诊断和解决该问题。

## 使用 tview

如果你从 tview 发送了 `d pos` 命令，有几种可能的情况。

**语法无效**：如果诊断通道回复 OK，那么这不是问题所在。如果它回复其他内容，那么错误信息会为你提供问题的线索。`d pos` 命令的语法可以在 [诊断协议参考手册]( http://localhost:8000/mjbots/moteus/protocol/diagnostic/#d-pos) 中找到。

**位置模式故障**：如果诊断通道回复 OK，那么要确定问题，你必须查看当前上报的模式和故障状态。它们将作为 `servo_stats` 遥测通道中的前两项出现。首次尝试控制时的常见问题包括：

 - "39 (outside bounds)"：`servopos.position_min` 和 `servopos.position_max` 要么没有配置，要么电机的起始位置在配置的边界之外。将其中任何一个设置为 `nan` 都会禁用它。
 - "33 (gate driver fault)"：这表示板载 MOSFET 门极驱动器报告了故障。要知道是哪一个，你需要展开 `drv8323` 遥测通道。唯一「容易」解决的问题是 `uvlo`，它表示 moteus 试图吸取超过电源所能提供的功率。
 - "43 (no position)"：此故障最常见的原因是控制器尚未校准。请参见 [快速入门中的校准部分](../guides/calibration.zh-CN.md)。

一旦确定了故障原因，可以通过发送以下命令来清除：

`d stop`

然后可以再次尝试相关命令。

## 从脚本或应用程序

如果你有一个 python 脚本、C++ 应用程序或其他工具在控制 moteus，而它没有运动，你可以使用以下几种工具。

**请求并显示模式和故障**：如果你的应用程序尚未请求和显示模式与故障，它应该这样做。

**记录 CAN-FD 数据**：如果应用程序无法修改或难以修改，你可以 (a) 如果它使用 python 库，使用 `--can-debug OUT.LOG` 命令标志保存所有发送和接收帧的记录，或 (b) 在总线上连接第二个 CAN-FD 适配器并用它来记录数据。

获得数据后，可以使用 `utils/decode_can_frame.py` 对其进行检查，以查看 moteus 上报的内容。

问题可能与上文「使用 tview」部分相同。

使用寄存器协议（python 和 C++ 库所使用的协议）的应用程序还有一个常见问题：看门狗超时（watchdog timeout）。对于寄存器协议，默认情况下 moteus 要求命令至少每 100ms 到达一次，否则它会进入模式 11，即超时（timeout）模式。一旦进入此模式，必须发送「stop」命令才能清除它。

看门狗定时器的配置在 [参考手册](../reference/configuration.zh-CN.md#servodefault_timeout_s) 中有描述。
