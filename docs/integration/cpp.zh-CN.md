# C++ 客户端库

moteus 提供了一个 C++ 库，可用于通过受支持的 CAN-FD 适配器命令和控制 moteus 控制器。

## 集成（Integration）

moteus C++ 库是仅头文件（header only）的，因此有几种可选方案可将 C++ 库集成到您的项目中。

**复制源文件**：一种方案是将所有相关的头文件复制到您的源码树中：

- [https://github.com/mjbots/moteus/tree/main/lib/cpp/mjbots/moteus](https://github.com/mjbots/moteus/tree/main/lib/cpp/mjbots/moteus)

**CMake**：存在一个顶层的 `CMakeLists.txt` 文件，可与 [CMake](https://cmake.org) 的 `FetchContent` 一起使用，将头文件引入到您的项目中。

```
include(FetchContent)
FetchContent_Declare(
  moteus
  GIT_REPOSITORY https://github.com/mjbots/moteus.git
  # Pin to a specific commit for a reproducible build.  The trailing
  # comment names the corresponding C++ release tag for readers.
  GIT_TAG        a1b2c3d4e5f6789012345678901234567890abcd  # cpp/v1.0.0
)

FetchContent_MakeAvailable(moteus)

add_executable(myproject myproject.cc)
target_link_libraries(myproject moteus::cpp)
```

请固定到提交哈希（commit hash）而不是标签本身。原则上标签是可变的（它们可以被强制推送或重新创建），因此哈希才是保证可复现获取的唯一引用。`# cpp/vX.Y.Z` 结尾注释使版本保持可读，并且是您在升级到新版本时——连同哈希一起——需要更新的内容。

最新的 C++ 版本及其提交哈希列在[发布（Releases）页面](https://github.com/mjbots/moteus/releases)上——在页面中键入 `cpp/` 进行筛选，即可仅查看 C++ 版本。

**bazel**：moteus 也确实导出了 [bazel](https://bazel.build) 的 `BUILD` 文件，并且也可以通过该机制集成到 bazel 项目中。

## 用法（Usage）

基本用法与 [python](python.zh-CN.md) 库相似，但略有不同。下面是一个最小示例：

```cpp
#include <iostream>
#include <unistd.h>
#include "moteus.h"

namespace moteus = mjbots::moteus;

int main(int argc, char** argv) {
  moteus::Controller::DefaultArgProcess(argc, argv);

  moteus::Controller c([]() {
    moteus::Controller::Options options;
    options.id = 1;
    return options;
  }());

  moteus::PositionMode::Command command;
  command.position = std::numeric_limits<double>::quiet_NaN();

  while (true) {
    const auto maybe_result = c.SetPosition(command);
    if (maybe_result) {
      const auto& v = maybe_result->values;
      std::cout << "Mode: " << v.mode
                << " Fault: " << v.fault
                << "Position: " << v.position
                << " Velocity: " << v.velocity
                << "\n";
    }
    ::usleep(10000);
  }
  return 0;
}
```

## 指定替代查询（query）寄存器

默认情况下，库只查询寄存器的子集。

### 常用寄存器选择

要选择其他常用选项，您可以 (a) 更改默认查询解析（query resolution），或 (b) 传入一个“覆盖（override）”查询解析。

### 选项 (a)：更改默认值

```cpp
moteus::Controller c([]() {
  moteus::Controller::Options options;
  options.query_format.power = moteus::kFloat;
  return options;
}());
```

### 选项 (b)：指定一个“覆盖”

```cpp
moteus::Controller c;
moteus::PositionMode::Command command;
moteus::Query::Formay query_override;

query_override.power = moteus::kFloat;
c.SetPosition(command, nullptr, &query_override);
```

### 不太常用的寄存器选择

对于在 `Query::Format` 中没有预定义字段的寄存器，您可以通过构造函数选项或覆盖参数使用 `extra` 机制。

```cpp
moteus::Controller c([]() {
  moteus::Controller::Options options;

  options.query_format.extra[0].register_number = moteus::Register::kAux1AnalogIn1;
  options.query_format.extra[0].resolution = moteus::kFloat;

  return options;
}());
```

## 指定替代命令（command）寄存器

与查询一样，默认情况下，只有寄存器子集会随 `SetPosition` 或 `MakePosition` 命令发送。在未做额外处理时，只会发送 "position" 和 "velocity"。要选择其他寄存器，可以通过 (a) 默认值更改它们，或通过 (b) 覆盖它们。

### 选项 (a)：更改默认值

```cpp
moteus::Controller c([]() {
  moteus::Controller::Options options;
  options.position_format.feedforward_torque = moteus::kFloat;
  return options;
}());

moteus::PositionMode::Command command;
command.position = std::numeric_limits<double>::quiet_NaN();
command.velocity = 0.0;
command.feedforward_torque = 0.1;

c.SetPosition(command);
```

### 选项 (b)：指定一个“覆盖”

```cpp
moteus::Controller c;

moteus::PositionMode::Command command;
command.position = std::numeric_limits<double>::quiet_NaN();
command.velocity = 0.0;
command.feedforward_torque = 0.1;

moteus::PositionMode::Format format;
format.feedforward_torque = moteus::kFloat;

c.SetPosition(command, &format);
```

## 替代用法模式（Alternate Usage Modes）

与 Python 库一样，C++ 库有两种操作模式。一种旨在便于使用，另一种旨在最大化总线利用率和命令速率（command rate）。前者是上文使用的 `Set*` 变体方法。与 Python 一样，也存在 `Make*` 变体方法，它们生成帧但不发送。这些方法可与手动构造的传输（transport）一起使用。

```cpp
auto transport = moteus::MakeSingletonTransport();

moteus::Controller c([&]() {
  moteus::Controller::Options options;
  options.transport = transport;
  return options;
}());

std::vector<moteus::CanFdFrame> commands_to_send;
std::vector<moteus::CanFdFrame> replies_to_receive;

moteus::PositionMode::Command command;
commands_to_send.push_back(c.MakePosition(command));

transport->BlockingCycle(
  commands_to_send.data(),
  commands_to_send.size(),
  &replies_to_receive);
```
