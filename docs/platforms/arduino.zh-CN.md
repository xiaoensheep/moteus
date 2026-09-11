# Arduino

moteus 为与 Arduino 兼容的微控制器（microcontroller）系统提供了一个简化版的 C++ 库。

## 安装

打开 Arduino 库管理器，搜索 "moteus" 并安装。然后打开其中一个示例并修改它。"WaitComplete" 是一个不错的入门示例。

## 支持的硬件 ##

**微控制器**：几乎任何与 Arduino 软件兼容的开发板都受支持

**CAN-FD**：该库支持：

 * 与 [acan2517FD 库](https://github.com/pierremolinaro/acan2517fd) 兼容的 CAN-FD 控制器与收发器（transceiver）
 * Teensy 4.x 开发板上的板载 ACAN_T4 外设
 * STM32 FDCAN HAL 外设

**UART**：该库几乎可以从任何微控制器驱动配置为 UART 服务器的 moteus。

### CAN-FD: ACAN2517FD

有两处配置因开发板和控制器而异。

#### 引脚分配

`ACAN2517FD` 构造函数要求传入连接到 MCP2517FD 控制器的 SPI 外设所使用的正确引脚。对于外部控制器，你可以使用物理连接到控制器的引脚。对于集成的 MCP2517FD，你需要查阅你的开发板文档。

```cpp
#define MCP2517_CS  17
#define MCP2517_INT 7
// For the Longan CANBed FD, the "SPI" peripheral determines which
// pins are used.
ACAN2517FD can(MCP2517_CS, SPI, MCP2517_INT);
```

#### CAN-FD 时序

你需要匹配你的适配器（adapter）所使用的 CAN-FD 基础时钟速率。不同的适配器有不同的基础时钟速率。

**Longan Labs CANBed FD**：该开发板使用 20MHz 时钟

**Mikro MCP2517FD Click**：该开发板默认使用 40MHz 时钟，但可以通过跳线进行配置。

无论哪种情况，你都需要使用 `ACAN2517FDSettings` 对象传入正确的时钟速率。

```cpp
ACAN2517FDSettings settings(
    ACAN2517FDSettings::OSC_20MHz,
    1000ll * 1000ll,
    DataBitRateFactor::x1);

settings.mArbitrationSJW = 2;
settings.mDriverTransmitFIFOSize = 1;
settings.mDriverReceiveFIFOSize = 2;

const uint32_t errorCode = can.begin(settings, [] { can.isr(); });
```

### CAN-FD: Teensy 4.x - ACAN_T4

配置 Teensy 4.x 库只需要选择使用哪个 CAN-FD 外设。

```
MoteusTeensyCanFd canBus(ACAN_T4::can3)
```

其中 `can1`、`can2` 或 `can3` 都可以。

### CAN-FD: STM32 FDCAN HAL

STM32 FDCAN HAL 外设的集成目前尚无文档，但有一些源代码示例存在。

### UART

使用 UART 控制方式时，有两种选择：

1. Arduino 的 HardwareSerial 类
2. 用类似接口自行实现一个串口类

使用 Arduino 串口类看起来像这样：

```
using MyTransport = MoteusUart<HardwareSerial>;
MyTransport uart_bus(Serial1);
MoteusController<MyTransport> moteus1(uart_buf, []() {
  MoteusController<MyTransport>::Options options;
  return options;
}());
```

实现自定义串口类需要以下方法：

```
class MySerial {
  int available();
  int read();
  size_t write(const uint8_t* size_t);
};

using MyTransport = MoteusUart<MySerial>;
```

## 注意事项

从微控制器控制 moteus 时有一些注意事项。

### 仍需要连接操作系统的 CAN-FD 适配器

虽然你可以从 Arduino 库对 moteus 进行命令和控制，但目前还没有用控制器校准新电机的机制。因此，你 *必须* 有一个能够连接到装有操作系统的计算机的 CAN-FD 适配器，以便运行[校准过程](../guides/calibration.zh-CN.md)。使用这样的适配器配合 tview 来设置配置和调整 PID 增益也会容易得多。

### Teensy 4.x 和 STM32 FDCAN HAL 需要外部 CAN-FD 收发器

Teensy 4.x 内置的 CAN-FD 控制器以及各种 STM32 外设是逻辑电平外设。要在 CAN-FD 总线上工作，需要一个收发器来转换为线路电压。市面上有许多现成的选择，但要注意很多需要同时提供 5V 和 3V 电源。

### 终端电阻

许多外部 MCP2517FD 控制器/收发器没有终端电阻（termination resistor）。为了正确工作，CAN-FD 总线应在 CANL 和 CANH 之间放置 2 个 120 欧姆电阻，总线两端各一个。通常短总线只用一个终端电阻也可以勉强工作，但绝不能一个都不用。

### Flash 和存储限制

对于某些较小的 Arduino 平台，尤其是 Arduino Uno 或 Longan Labs CANBed FD，ACAN2517FD 库与 moteus 库的组合可能会占用相当大比例的 Flash 和 RAM。对于简单应用这不是问题，但如果你想执行更复杂的操作，最好使用性能更强的 Arduino 兼容处理器，比如 [Teensy 4](https://www.pjrc.com/store/teensy41.html) 或 [Nano Sense 33](https://store-usa.arduino.cc/products/nano-33-ble-sense-rev2)。

## 概述视频

可以在以下位置找到使用 Arduino 与 moteus 过程的视频概述：

{{ youtube("dbV0jIN8Ay4") }}
