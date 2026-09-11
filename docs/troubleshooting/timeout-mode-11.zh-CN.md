# moteus 报告模式 11 - 超时

在使用 tview 进行一些初步验证之后，也许你会将运动轨迹移植到 python 或 C++ 应用程序中以准备部署。然而，用户往往在此阶段开始看到 moteus 进入模式 11，即超时（timeout）模式，之后它会停止接受命令并开始抑制运动。本指南将告诉你为什么会发生这种情况以及如何修复。

## 原理

moteus 为通过 CAN-FD 到达的命令实现了「看门狗定时器」（watchdog timer）。一旦收到任何命令，下一个命令必须在某个时间段内到达。如果没有，moteus 就会进入超时模式。

此功能有以下几个原因：

1. **安全**：在实际部署中，moteus 可能被命令无限期地进行大幅或快速运动。如果你的应用程序崩溃，或承载它的计算机断电，你大概不希望 moteus 继续运行。
2. **便利**：在测试应用程序时，如果你退出应用程序，通常也希望 moteus 停止它正在做的一切。

一旦进入超时模式，moteus 只有在收到停止命令时才会退出该模式。这可以防止 moteus 意外恢复运行。

当使用诊断模式命令时（例如在 tview 的诊断通道控制台中输入命令），为方便起见，moteus 默认使用无限长的看门狗时长。这就是为什么切换到脚本后问题最常出现。python 和 C++ 库默认都使用更高效的寄存器模式命令。

## 修复方法

如果你遇到了超时，以下是一些解决方法：

### 禁用或延长超时时长

moteus 等待的时间长度可在以下位置配置：

```
servo.default_timeout_s
```

[(在参考手册中列于此)](../reference/configuration.zh-CN.md#servodefault_timeout_s)

要禁用它，请使用 `nan`，否则请指定以秒为单位的浮点数。

### 将你的应用程序组织为以固定间隔发送命令

为了保留看门狗的优势，你的应用程序需要持续向 moteus 发送命令。这意味着即使应用程序在等待时，它仍然需要发送命令。幸运的是，对于许多 moteus 命令，如果目标速度为 0，你可以反复发送相同的命令。

=== "Python"

    ```python
    # Don't do this:
    #  await asyncio.sleep(5)

    # Instead, do this:
    start = time.time()
    while time.time() - start < 5:
      await c.set_position(velocity=0.0, query=True)
      await asyncio.sleep(0.02)
    ```

=== "C++"

    ```cpp
    // Don't do this.
    //  ::usleep(1000000);

    // Instead, do this:
    #include <chrono>

    double get_time() {
      using namespace std::chrono;
      return duration<double>(system_clock::now().time_since_epoch()).count();
    }

    int main() {
      // setup ...

      const auto start = get_time();
      while (get_time() - start < 5) {
        mjbots::moteus::PositionMode::Command cmd;
        cmd.velocity = 0.0;
        cmd.position = NaN;
        c.SetPosition(cmd);

        ::usleep(10000);
      }

      return 0;
    }
    ```

### 上述两者的组合

即使在重组你的应用程序以更频繁地发送命令之后，也可能仍有意义去增大（但不是禁用）默认的看门狗超时。
