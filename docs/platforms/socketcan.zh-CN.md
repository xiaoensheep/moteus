# socketcan

许多 CAN-FD 适配器可以与 Linux socketcan 子系统配合使用。本节介绍如何配置它们以用于 moteus：

## 位时序

要与 moteus 通信，必须为 socketcan 设备同时配置波特率（bitrate）*以及* 采样点（sample point）*以及* sjw/dsjw。以下命令适用于大多数设备：

```
ip link set can0 up type can \
  bitrate 1000000 dbitrate 5000000 \
  sjw 10 dsjw 5 \
  sample-point 0.666 dsample-point 0.666 \
  restart-ms 1000 fd on
```

如果 `sjw` 或 `dsjw` 选项过大，某些设备会报错。在这种情况下，请选择尽可能大的 `sjw` 或 `dsjw` 选项。

## 不兼容的适配器

使用 slcan 协议的 CAN-FD 适配器，或者源自 canable 硬件的适配器（无论它们使用的是 slcan 还是 candlelight 固件），已知目前与 moteus 控制器不兼容。
