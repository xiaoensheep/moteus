# CAN-FD

命令和控制（command and monitor）moteus 并非必须使用 Python 和 C++ 库，它们只是让工作更轻松。任何能够以合适的时序生成和接收 CAN-FD 帧的系统都可以使用。以下是一些使其更容易的提示。

## 位时序（Bit timing）与基本通信

Moteus 要求 1Mbps/5Mbps 的 CAN-FD 时序，采样点（sample point）为 0.666，并且当 SJW 和 DSJW 设置尽可能大时效果最佳。

如果通信无法正常工作，建议在同一总线上同时接入一个 mjcanfd-usb-1x 与您的主机，并使用它来捕获帧。任何串口应用程序（或在 Linux 上使用 `cat`）都可以用于读取输出。由于它们以与 moteus 相同的时序启动，因此您可以用它来确认您的帧是否已被接收并得到响应。

## 解码总线上观察到的帧

moteus 仓库中存在一个工具，可以解码发送给 moteus 或由 moteus 发出的 CAN-FD 帧，这在诊断您发送了什么以及 moteus 为何响应或不响应时非常宝贵。首先，捕获相关 CAN-FD 帧的十六进制格式转储。然后：

```
moteus$ ./utils/decode_can_frame.py 1100110F
11 - READ_REGISTERS - INT8 1 registers
  00 - Starting at reg 0x000(MODE)
11 - READ_REGISTERS - INT8 1 registers
  0f - Starting at reg 0x00f(FAULT)
```

以及相应的响应：

```
moteus$ ./utils/decode_can_frame.py 210000210F00
21 - REPLY - 0 1 registers
  00 - Starting at reg 0x000(MODE)
   00 - Reg 0x000(MODE) = 0(STOPPED)
21 - REPLY - 0 1 registers
  0f - Starting at reg 0x00f(FAULT)
   00 - Reg 0x00f(FAULT) = 0
```


## 生成合适的命令帧

[CAN-FD 参考](../protocol/can.md)和[寄存器（register）参考](../protocol/registers.md)包含了生成和解析帧所需的信息。但是，这些内容可能非常庞杂。您可以使用 [Python 库](../integration/python.md)快速获取完成给定任务的示例帧。首先，创建一个能够实现您所需功能的 Python 脚本。然后，在交互式 Python REPL 中：

```
>>> print(c.make_position(position=1, velocity=2, query=True).data.hex())
01000a0e200000803f0000004011001f01130d
```

其中 `make_position` 的参数对应于您想要完成的操作。您可以使用 `decode_can_frame.py` 来验证此帧的内容：

```
moteus$ ./utils/decode_can_frame.py 01000a0e200000803f0000004011001f01130d
01 - WRITE_REGISTERS - 0 1 registers
  00 - Starting at reg 0x000(MODE)
   0a - Reg 0x000(MODE) = 10(POSITION)
0e - WRITE_REGISTERS - 3 2 registers
  20 - Starting at reg 0x020(COMMAND_POSITION)
   0000803f - Reg 0x020(COMMAND_POSITION) = 1.0
   00000040 - Reg 0x021(COMMAND_VELOCITY) = 2.0
11 - READ_REGISTERS - INT8 1 registers
  00 - Starting at reg 0x000(MODE)
1f - READ_REGISTERS - F32 3 registers
  01 - Starting at reg 0x001(POSITION)
13 - READ_REGISTERS - INT8 3 registers
  0d - Starting at reg 0x00d(VOLTAGE)
```
