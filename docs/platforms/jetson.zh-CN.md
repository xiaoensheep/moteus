# Nvidia Jetson 设置

许多（也许是全部）NVIDIA Jetson 开发板都内置了一个 CAN-FD 控制器。然而，很少有 NVIDIA Jetson 载板（carrier board）包含 CAN-FD 收发器（transceiver）。前者管理逻辑电平的 CAN-FD 协议，后者负责转换为电气线路接口，两者都是必需的。

从 NVIDIA Jetson 开发板与 moteus 控制器通信有三种主要选择：

1. **USB 适配器（Adapter）**：可以使用 mjcanfd-usb-1x 或类似的 USB 适配器。这种方式简单且相对方便，但体积较大，并且在存在物理振动或 EMI 的系统中可能不够可靠。

2. **PCIe/m.2 适配器**：许多 Jetson 开发板都有一个可用的 m.2 插槽。这些可以与诸如 [PEAK m.2 CAN-FD 适配器](https://www.peak-system.com/PCAN-M-2.473.0.html?&L=1) 之类的适配器配合使用。请注意，有些 Jetson 载板虽然带有 m.2 插槽，但其物理安装方式会导致无法使用 PEAK 适配器，因为会产生机械干涉。

3. **板载 Jetson 控制器**：要使用这种方式，你或者需要一个带 CAN-FD 收发器的载板，或者必须将外部收发器连接到相应的引脚。

对于选项 2 或 3，你可以使用与任何 socketcan 设备相同的方法来配置 CAN-FD 接口。该参考资料在这里：

- [socketcan 配置](../platforms/socketcan.zh-CN.md)
