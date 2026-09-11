# 软件安装（Software Installation）

本指南介绍如何安装与 moteus 控制器通信和配置所需的软件工具。

## 安装 moteus_gui

moteus_gui 软件包提供了必要的工具，包括 `tview`（遥测查看器）和 `moteus_tool`（配置实用工具）。安装方式因平台而略有不同。

=== "Linux"

    ```bash
    python -m pip install moteus-gui
    ```

    **注意**：在 Linux 上，你可能需要使用[虚拟环境](https://docs.python.org/3/library/venv.html#creating-virtual-environments)，或者 `--break-system-packages` 选项。

    **fdcanusb udev 规则**：如果你使用 fdcanusb 或 mjcanfd-usb-1x 适配器，可能需要设置 udev 规则，以便普通用户能够访问设备。请按照以下位置的说明操作：[https://github.com/mjbots/fdcanusb/blob/master/70-fdcanusb.rules](https://github.com/mjbots/fdcanusb/blob/master/70-fdcanusb.rules)

=== "Windows"

    ```bash
    python -m pip install moteus-gui
    ```

    **注意**：在某些较旧的 Windows 安装上，你需要使用 `python3` 而不是 `python` 来调用 python。

=== "macOS"

    ```bash
    python -m pip install moteus-gui
    ```

=== "Raspberry Pi（仅命令行）"

    ```bash
    python -m venv moteus-venv --system-site-packages
    ./moteus-venv/bin/pip install moteus

    ```

    **注意**：有关包含 GUI 和硬件配置在内的完整 Raspberry Pi 设置说明，请参阅[Raspberry Pi 设置](../platforms/raspberry-pi.zh-CN.md)。

## 运行 tview

tview 是用于配置和检查 moteus 控制器状态的主要工具。使用以下命令启动它：

```bash
python3 -m moteus_gui.tview
```

**替代方式**：你的 pip 安装可能已将 `tview` 脚本添加到 PATH 中，你也可以直接使用它。

**UART**：要通过逻辑电平 UART 使用 tview，必须手动指定串口设备：

=== "Linux"

    ```bash
    python3 -m moteus_gui.tview --fdcanusb /dev/serial/by-id/usb-FTDI_FT231X_USB_UART_DP050AG5-if00-port0
    ```

=== "Windows"

    ```bash
    python3 -m moteus_gui.tview --fdcanusb COM2
    ```

有关 UART 所需的连接器和线缆信息，请参阅 [UART 集成指南](../integration/uart.zh-CN.md)。

## 了解 tview

tview 有三个主窗格：左、右和底部。

![](images/tview-overview.png)

### 左窗格选项卡

左窗格包含两个选项卡：

**右侧选项卡（默认）**：显示控制器当前报告的所有遥测项的层级树。

**左侧选项卡**：显示所有可配置参数的层级树。你可以双击参数值并输入新值来更新它。

### 右窗格

右窗格显示遥测项的实时绘图。你可以通过在遥测选项卡（左窗格，右侧选项卡）中右键单击遥测项来将绘图添加到其中。

### 底部窗格

底部窗格包含两个选项卡：

**左侧选项卡 - 控制台（默认）**：一个显示 moteus 诊断协议的命令行控制台。它：

- 显示 tview 内部发送的命令及其响应
- 提供一个交互式控制台，使用[诊断协议](../protocol/diagnostic.zh-CN.md)与设备交互

**右侧选项卡 - Python**：一个 Python REPL，可用于使用 moteus Python 库与控制器交互。

## 后续步骤

现在你已安装软件并了解 tview 界面，请继续进行[配置](configuration.zh-CN.md)指南，为你的具体电机和应用设置 moteus 控制器。
