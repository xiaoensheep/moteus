# UART

在没有 CAN-FD 可用的系统中，可以通过 TTL 电平的 UART 连接命令和监控 moteus。

## 默认配置

默认情况下，固件版本足够新的 moteus 控制器会为上述用途配置以下引脚。

* [moteus-r4](../reference/pinouts.md#moteus-r4)：aux2 - JST ZH-4
  * A - 主机 TX / moteus RX
  * B - 主机 RX / moteus TX
* [moteus-c1](../reference/pinouts.md#moteus-c1)：aux2 - JST GH-7
  * C - 主机 RX / moteus TX
  * D - 主机 TX / moteus RX
* [moteus-n1](../reference/pinouts.md#moteus-n1)/[moteus-x1](../reference/pinouts.md#moteus-x1)：aux1 - JST GH-8
  * D - 主机 TX / moteus RX
  * E - 主机 RX / moteus TX

串口参数：

* 波特率（Baudrate）：921600
* 数据位（Data bits）：8
* 停止位（Stop bits）：1
* 校验（Parity）：无（None）

要将这些引脚用于任何其他用途，必须将相应的 [`aux[12].uart.mode`](../reference/configuration.md#aux12uartmode) 设置为 "Disabled (0)"。

## 协议（Protocol）

在此模式下，moteus 表现得就像它是 UART 上的一个 fdcanusb 设备。如果使用操作系统中 UART 的设备名来代替 fdcanusb，则可以直接使用 moteus 的 Python、C++ 和 rust 库。唯一的区别是，moteus 允许在命令行中包含校验和（checksum），并且始终在响应行中输出校验和。一旦任何命令带校验和发送过，此后所有命令都必须带校验和。

校验和使用以下格式：

```
data line *XX
```

其中 '*' 字符后跟两个十六进制数字。这些是使用多项式 0x97 的 CRC-8。示例计算程序如下：

=== "Python"

    ```python
    _CRC8_TABLE = [
        0x00, 0x97, 0xb9, 0x2e, 0xe5, 0x72, 0x5c, 0xcb,
        0x5d, 0xca, 0xe4, 0x73, 0xb8, 0x2f, 0x01, 0x96,
    ]

    def compute_crc8(data: bytes) -> int:
        """Compute CRC-8 using polynomial 0x97, nybble-at-a-time."""
        crc = 0
        for b in data:
            crc = _CRC8_TABLE[((crc >> 4) ^ (b >> 4)) & 0x0f] ^ ((crc << 4) & 0xff)
            crc = _CRC8_TABLE[((crc >> 4) ^ (b & 0x0f)) & 0x0f] ^ ((crc << 4) & 0xff)
        return crc

    ```

=== "C++"
    ```cpp
    static constexpr uint8_t kCrc8Table[16] = {
        0x00, 0x97, 0xb9, 0x2e, 0xe5, 0x72, 0x5c, 0xcb,
        0x5d, 0xca, 0xe4, 0x73, 0xb8, 0x2f, 0x01, 0x96,
    };

    static uint8_t ComputeCrc8(const char* data, size_t len) {
      uint8_t crc = 0;
      for (size_t i = 0; i < len; i++) {
        const uint8_t b = static_cast<uint8_t>(data[i]);
        crc = kCrc8Table[((crc >> 4) ^ (b >> 4)) & 0x0f] ^ (crc << 4);
        crc = kCrc8Table[((crc >> 4) ^ (b & 0x0f)) & 0x0f] ^ (crc << 4);
      }
      return crc;
    }
    ```

## 与 moteus 工具和库一起使用

要通过 UART 连接使用 moteus_tool 或 tview，您必须在命令行上指定 UART 设备的路径：

```bash
python -m moteus_gui.tview --fdcanusb /dev/ttyUSB0
```

或者：

```bash
python -m moteus.moteus_tool --fdcanusb /dev/ttyUSB0 -t 1 --info
```

在与 Python 库集成时，您必须手动构造一个传输（transport）：

```python
fdcanusb = moteus.Fdcanusb('/dev/ttyUSB0')
c = moteus.Controller(id=1, transport=fdcanusb)
```

或对于 C++：

```cpp
using mjbots;
moteus::Controller controller([]() {
  moteus::Controller::Options options;
  options.transport = std::make_shared<moteus::Fdcanusb>("/dev/ttyUSB0");
  return options;
}());
```

或对于 rust：

```rust
let transport = moteus::Fdcanusb::open("/dev/ttyUSB0")?;
let mut controller = moteus::BlockingController::with_transport(1, transport);
```

对于嵌入式（no_std）rust 主机，`moteus-protocol` crate 在其 `fdcanusb` 模块中为此协议提供了一个免分配（allocation-free）的编解码器（codec）。

有关与 Arduino 一起使用的说明，请参阅该[平台集成参考](../platforms/arduino.zh-CN.md)。

## 注意事项（Caveats）

### 重新配置（Reconfiguring）

如果 UART 是唯一的通信方式，那么将 UART 引脚配置到不同的端口或不同的波特率可能会很有挑战性。与几乎所有 moteus 可配置值一样，配置更改会*立即*生效。因此，建议首先配置新的端口位置。如果新端口位于编号较小的 aux 端口上，那么一旦它被配置，它将成为唯一的控制手段。然后，主机可以切换到该端口以保存配置，并重新配置原来的 UART 引脚。

类似地，如果某个 aux 端口配置存在错误，那么该错误将禁用 UART 控制以及该 aux 端口上的所有其他功能，即使错误与该 aux 端口上的另一个引脚相关联。可能需要给设备断电重启（power cycle）或使用 CAN-FD 来恢复控制。

**注意（Note）**：如果将波特率从默认的 921600 更改，并且使用了 moteus 客户端工具，则可以使用 `--fdcanusb-baudrate` 选项指定正确的值。参见[客户端工具参考](../reference/client-tools.md#configuring-the-communications-transport)

### 寻址（Addressing）

moteus 模拟（emulate）了 fdcanusb 协议的一个子集，包括任何已配置的 ID 和 CAN 前缀（prefix）。因此，如果您更改 `id.id` 或 `can.prefix`，则在后续与该设备的通信中需要使用正确的 CAN ID 和前缀。

### 非总线拓扑（Non-bus topology）

使用 UART 控制时，单个主机 UART 端口只能连接一个 moteus 设备。

### 刷写（Flashing）

无法使用 UART 传输来刷写新固件。仅支持 CAN-FD 以及带有 st-link 的 SWD 端口。
