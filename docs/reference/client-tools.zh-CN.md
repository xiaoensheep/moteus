# 客户端工具

## tview 用法

tview 可以同时监控和控制 1 个或多个设备。可以通过以下命令启动：

```
python3 -m moteus_gui.tview --target 1[,2,3]...
```

运行时，每个设备的配置都可以在左侧标签页中修改，实时遥测（telemetry）值可以在右侧标签页中显示或绘制。诊断（diagnostic）模式命令可以在底部终端窗口中输入。

### 与特定设备通信

如果有多个设备可用，可以通过在命令前加上 `ID>` 前缀来向特定设备发送命令。例如，要向 ID #2 发送停止命令，可以这样做：

```
2>d stop
```

字符 'A' 可用于同时向所有设备发送命令。

```
A>d stop
```

### 一次发送多条命令

`&&` 标记可用于分隔单独的命令，这些命令会依次连续发出。例如：

```
1>d stop && 2>d stop
```

会向两个设备都发送该命令。

### 延时

可以通过输入以冒号（':'）字符为前缀的整数毫秒数，在一系列命令中插入延时。例如：

```
d pos nan 0.5 1 s0.5 && :1000 && d stop
```

会命令进入位置模式（position mode），等待 1 秒，然后命令停止。

### 等待轨迹完成

可以使用 `?` 字符暂停一系列命令，直到控制器完成其轨迹（trajectory）。

```
d pos 0 0 nan a2 && ? && d pos 1 0 nan a2
```

会移动到位置 0，等待该运动完成，然后移动到位置 1，全程加速度限制为 2。

可以通过在问号后跟 ID 号来查询特定设备。

```
2>d pos 0 0 nan a2 && 1>d pos 10 0 0 nan a2 && ?2 && 2>d stop
```

## 校准

假设你的控制器已经安装了固件，你可以使用以下流程校准（calibrate）控制器。

```
python3 -m moteus.moteus_tool --target 1 --calibrate
```

警告：任何已连接的电机都必须能够自由旋转。它会在两个方向上高速旋转。

针对全新类型的电机完成校准后，你可能需要调整 PID 增益（gain），电机才会表现良好，并且/或者配置 `motor_position.rotor_to_output_ratio`。

## 设置“零位偏移”

moteus 控制器在重新上电后能够定位一个圈内的位置，并且启动时上报的位置会在 -0.5 到 0.5 之间。物理零位可以通过以下命令设置：

```
python3 -m moteus.moteus_tool --target 1 --zero-offset
```

## 配置通信传输方式

`moteus_tool` 和 `tview` 可以配置为通过多种传输方式与 moteus 控制器通信。默认情况下，它会尝试自动检测 fdcanusb 或 socketcan 接口。

要强制使用特定的 fdcanusb 或逻辑电平 UART，可以使用：

```
python3 -m moteus.moteus_tool --fdcanusb /path/to/fdcanusb
```

要强制使用特定的 python-can 方法，可以使用：

```
python3 -m moteus.moteus_tool --can-iface socketcan --can-chan can0
```

其中 `--can-iface` 指定 python-can 的“接口”（interface），`--can-chan` 指定“通道”（channel）。

注意，这些是在任何其他可能需要用到的 `moteus_tool` 或 `tview` 选项之外的附加选项。

如果逻辑电平 UART 使用了非默认波特率，可以通过 `--fdcanusb-baudrate` 指定波特率，例如：

```
python3 -m moteus.moteus_tool --fdcanusb /dev/ttyUSB0 --fdcanusb-baudrate 115200
```
