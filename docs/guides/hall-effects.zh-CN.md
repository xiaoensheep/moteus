# 带霍尔效应传感器的电机（Motor with Hall Effect Sensors）

许多电机都包含霍尔效应传感器，用于向电机控制器提供换相信息。moteus 能够将霍尔效应传感器同时用于换相和输出定位。首先是一些注意事项：

**可能并不需要它们**：仅仅因为电机内置了霍尔效应传感器，并不意味着你必须使用它们。如果你可以通过将 moteus 相对转子适当安装来使用集成到 moteus 中的同轴磁编码器，那将始终比使用霍尔效应编码器获得更好的结果。

**你可能需要额外的硬件**：大多数使用霍尔效应的配置都需要额外的滤波电容，放置在 moteus 附近，位于每条信号线与地之间。建议从 1nF 开始。它们可以加在线束中，也可以加在 moteus-c1、moteus-n1 和 moteus-x1 上未安装元件的 0402 焊盘上。

**用于输出定位的霍尔效应传感器在低速下性能非常差**：虽然霍尔效应传感器对换相和较高速输出运行来说勉强可用，但当用作低速输出编码器时，它们提供的性能会很差。这意味着许多位置控制应用（本质上常常是低速的）会得到较差的结果。

**moteus-r4 不易驱动许多霍尔效应传感器**：具体而言，moteus-r4 只有 3.3V 电源供外部设备使用，而大多数霍尔效应编码器需要 5V。此外，将霍尔效应传感器连接到 moteus-r4 的唯一方式是使用电路板背面未安装元件的焊盘，并且*不能*使用板载编码器。其他 moteus 产品在电源电压和连接器方面没有这些规格限制。

如果仍然希望使用霍尔效应进行换相或输出定位，下面的配置可以帮你入门。

## aux2 配置 ##

此配置使用连接到 aux2 GH7 连接器的霍尔效应传感器，同时用于换相和输出定位。

moteus-c1 [(c1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-c1)、moteus-n1 [(n1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-n1) 和 moteus-x1 [(x1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-x1) 都有一个 aux2 GH7 连接器，可提供 5V 输出，并且具有 5V 耐压的 IO 引脚。典型的接线图如下所示，不过霍尔连接器是“名义上的”，因为并不存在真正的标准：

![](images/hall-wiring.png)

如上所述，应在每个信号线与地之间的线束中安装 1nF 电容，或者安装在电路板上相应的未安装元件焊盘上。

应设置以下 aux2 配置参数。

```
aux2.pins.0.mode 6  # hall
aux2.pins.0.pull 1  # pull_up
aux2.pins.1.mode 6  # hall
aux2.pins.1.pull 1  # pull_up
aux2.pins.2.mode 6  # hall
aux2.pins.2.pull 1  # pull_up
aux2.hall.enabled 1
aux2.uart.mode 0    # disable serial control
```

然后按如下方式配置 motor_position：

```
motor_position.sources.0.aux_number 2
motor_position.sources.0.type 4  # hall
```

最后，校准时必须指定极数。

```
python -m moteus.moteus_tool -t 1 --calibrate --cal-motor-poles 30
```

## 高级：双编码器设置

性能更好的配置可以使用霍尔效应传感器进行换相，并使用另一个编码器测量输出位置。这里不直接描述该配置，但你可以查看[双编码器](dual-encoders.zh-CN.md)指南以及[编码器参考](../reference/encoders.zh-CN.md)来了解如何配置它。
