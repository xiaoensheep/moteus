# 双编码器配置（Dual Encoder Configuration）

moteus 常用于在转子和最终被控对象之间使用齿轮箱或其他减速器的应用。在这些情况下，配备一个测量减速器输出的辅助编码器会很有价值：

**绝对定位**：如果转子上只有一个编码器，就无法绝对地知道输出在何处。

**更高的精度或传感性能**：moteus 自带的板载编码器对几乎所有电机的换相而言都足够了，但某些应用可能需要更低的位置或速度传感噪声，或更高的分辨率，以实现精确的极低速运动或非常精确的定位。

本节将介绍如何设置几种常见的双编码器配置。

## <a name="match"></a>配置双编码器时的重要因素

1. **符号匹配**：moteus 要求所有编码器的符号一致。这些必须在 `motor_position.sources.X.sign` 中手动配置。为验证符号是否正确，绘制所有已配置编码器的 `motor_position.sources.X.filtered_value`，用手转动系统，并验证所有绘图沿同一方向移动。

2. **偏移匹配**：如果多个编码器的绝对值必须相互关联（例如用于去歧义配置），则必须匹配偏移，使这些编码器在同一时刻读取为 0。这些偏移必须在 `motor_position.sources.X.offset` 中手动配置。为验证其正确性，绘制所有已配置编码器的 `motor_position.sources.X.filtered_value`，用手将系统转到零点，并验证所有编码器都读取为 0。

## 板载 + 8x 减速器 + 同轴 MA600 - 去歧义

在此配置中，转子由 moteus 板载编码器传感，输出由同轴 MA600 编码器传感。假设回程间隙较小，输出定位由板载编码器完成，输出编码器仅用于去歧义。这样可以获得最佳性能，因为板载编码器的有效分辨率会乘以齿轮减速比，将远优于输出端的 MA600。

MA600 连接到以下控制器之一的 aux2 端口：moteus-c1 [(c1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-c1)、moteus-n1 [(n1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-n1) 或 moteus-x1 [(x1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-x1)。

如果使用 [mjbots MA600 转接板](https://mjbots.com/products/ma600-breakout)附带的线缆，即可实现此引脚连接。

如果从出厂默认配置开始，请先配置 aux2 端口：

```
aux2.pins.0.mode 1  # spi
aux2.pins.1.mode 1  # spi
aux2.pins.2.mode 1  # spi
aux2.pins.3.mode 2  # spi_cs
aux2.spi.mode 5     # ma600
aux2.spi.rate_hz 6000000
aux2.uart.mode 0    # disable serial control
```

然后配置 motor_position：

```
motor_position.sources.1.aux_number 2
motor_position.sources.1.type 1
motor_position.sources.1.cpr 65536
motor_position.sources.1.reference 1  # output
motor_position.rotor_to_output_ratio 0.125
motor_position.output.reference_source 1
```

MA600 的符号和偏移需要与板载编码器匹配，参见[上面的章节](#match)。

## 板载 + 10x 减速器 + 输出端 AksIM-2

在此配置中，板载编码器用于传感转子，而来自 RLS 的 AksIM-2 编码器用于输出端。AksIM-2 的精度和噪声性能远优于板载同轴编码器，因此此配置可用于需要非常精确的定位或非常精确低速运行的应用。

AksIM-2 连接到 moteus-n1 [(n1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-n1) 或 moteus-x1 [(x1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-x1) 上的 GH-6 RS422 连接器。

moteus 兼容按如下方式配置的 AksIM-2 编码器：

- **通信接口**：SF - 异步串行、RS422、5V
- **通信协议变体**：F - 1000 kbps
- **分辨率**：<= 19B，不支持多圈

如果从出厂默认配置开始，请先配置 aux1 端口。

```
aux1.pins.3.mode 3 # uart
aux1.pins.4.mode 3 # uart
aux1.uart.mode 1 # aksim2
aux1.uart.baud_rate 1000000
aux1.rs422 1
```

然后配置 motor_position：

```
motor_position.rotor_to_output_ratio 0.1
motor_position.sources.1.aux_number 1
motor_position.sources.1.type 2          # uart
motor_position.sources.1.cpr 4194304     # used by moteus for all AksIM-2
motor_position.sources.1.reference 1     # output
motor_position.output.source 1
```

AksIM-2 的符号和偏移需要与板载编码器匹配，参见[上面的章节](#match)。

## 霍尔 + 10x 减速器 + 离轴 MA600

在此配置中，使用霍尔效应编码器来传感换相，并使用 MA600 配合一个径向磁化的环形磁铁来进行输出定位。它是空心轴带减速器执行器的最低成本方案之一，因为离轴 MA600 的性能以及霍尔效应传感器的性能都只是勉强可用。

MA600 连接到 moteus-n1 [(n1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-n1) 或 moteus-x1 [(x1 引脚定义)](../reference/pinouts.zh-CN.md#moteus-x1) 的 AUX1，霍尔效应传感器连接到 AUX2。

首先，我们将配置霍尔效应传感器并对其进行校准。

```
aux2.pins.0.mode 6  # hall
aux2.pins.0.pull 1  # pull_up
aux2.pins.1.mode 6  # hall
aux2.pins.1.pull 1  # pull_up
aux2.pins.2.mode 6  # hall
aux2.pins.2.pull 1  # pull_up
aux2.hall.enabled 1
motor_position.rotor_to_output_ratio 0.1
motor_position.sources.0.aux_number 2
motor_position.sources.0.type 4  # hall
```

然后，对霍尔效应传感器进行校准：

```
python -m moteus.moteus_tool -t 1 --calibrate --cal-motor-poles 30
```

现在我们将配置 MA600。

```
aux1.pins.0.mode 1  # spi
aux1.pins.1.mode 1  # spi
aux1.pins.2.mode 1  # spi
aux1.pins.3.mode 2  # spi_cs
aux1.spi.mode 5     # ma600
aux1.spi.rate_hz 6000000
aux1.uart.mode 0    # disable serial control
motor_position.sources.1.aux_number 1
motor_position.sources.1.type 1  # spi
motor_position.sources.1.cpr 65536
motor_position.sources.1.reference 1  # output
```

然后，在将 MA600 配置为输出源之前，按照[这里的步骤](#match)匹配 MA600 和霍尔效应的符号。

对于离轴 MA600，应执行 BCT 整定，还可以执行非线性补偿。

```
moteus$ ./utils/measure_ma732_bct.py
```

要执行非线性补偿，可以使用：

```
moteus$ ./utils/compensate_encoder.py
```

最后，可以将 MA600 配置为输出源：

```
motor_position.output.source 1
```
