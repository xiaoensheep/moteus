# 烧录与构建固件

moteus 控制器既有可升级的固件，也有宽松许可的固件源代码。本节描述如何烧录新（或旧）版本的固件，以及如何从源代码构建它。

## 通过 CAN 烧录

最新固件可从以下地址下载：[https://github.com/mjbots/moteus/releases](https://github.com/mjbots/moteus/releases)

每个固件发布附带两个 ELF：

- `moteus-fw-<version>+g<sha>.elf` — 应用固件。**这是你正常烧录所需的。**
- `moteus-bl-<version>+g<sha>.elf` — CAN 引导加载程序（bootloader）。仅在通过 SWD 端口进行高级引导加载程序恢复时需要（见下文）；引导加载程序已预装在每个 moteus 上，你通常不需要烧录它。

`<version>` 是 semver 发布版本（例如 `1.0.0` 或 `1.0.0-rc.1`），`<sha>` 是发布构建所基于提交的 10 位 git 短 SHA（semver 下的构建元数据——仅供参考）。

下载 `moteus-fw-...elf`，将其保存到计算机上某处，然后在以下命令中用其路径替换 `path/to/file.elf`。

```
python3 -m moteus.moteus_tool --target 1 --flash path/to/file.elf
```


## 构建固件

要构建 moteus 固件，需要一个 x86-64 Ubuntu 22.04、24.04 或 26.04 系统。

首先安装依赖项：

```
sudo python3-build python3-can python3-serial python3-setuptools \
     python3-pyelftools python3-qtpy python3-wheel \
     python3-importlib-metadata python3-scipy python3-usb \
     mypy nodejs wget curl
```

然后可以使用以下命令构建固件：

```
tools/bazel build --config=target //:target
```

## 从 SWD 端口烧录

如果 CAN-FD 不可用，则可以使用每个 moteus 上的 SWD 连接器烧录新固件。首先安装依赖项：

```
sudo apt install openocd binutils-arm-none-eabi
```

然后将一个 [stm32 编程器](https://mjbots.com/products/stm32-programmer) 连接到计算机的 USB 端口和 moteus 上的 SWD 端口。

可以使用以下命令构建并烧录固件：

```
tools/bazel build --config=target //fw:flash
```

或者，如果已经构建，则使用以下命令烧录：

```
./fw/flash.py
```

该 python 脚本也可用于烧录预编译镜像：

```
./fw/flash.py path/to/firmware.elf path/to/bootloader.elf
```

## 从 SWD 端口调试

当 [stm32 编程器](https://mjbots.com/products/stm32-programmer) 连接到 SWD 端口时，可以使用 gdb 调试固件。首先，安装依赖项：

```
sudo apt install gdb-multiarch
```

然后，在一个终端中运行：

```
./run_openocd_noreset.sh
```

在另一个终端中运行：

```
gdb-multiarch -x moteus-debug.gdb bazel-out/stm32g4-opt/bin/moteus.elf
```

调试提示：

- **红色灯闪烁（Flashing Red Light）**：这意味着调试断言失败。使用 `bt` 获取回溯会告诉你调用位置。
- **池大小（Pool Size）**：固件在初始化期间从固定大小的池中分配数据。如果添加了新结构，此大小可能会被超出。要么减小结构的大小，要么增大池的大小。
- **优化（Optimizations）**：moteus 固件必须在全局启用优化。这使调试变得困难。一些策略：
 - 将你需要的数据暴露给 tview 可用的结构
 - 单步执行汇编
 - 对单个函数使用 `__attribute__((optimize("O0")))` 来禁用优化
