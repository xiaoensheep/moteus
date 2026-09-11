# 诊断协议

以下命令集用于调试和诊断。它可以从 `tview` 或 `multiplex_tool` / `moteus_tool` 的 `--console` 模式进入。

## `d` - 板级调试

### `d stop`

这会使控制器进入“已停止（stopped）”状态，该状态会禁用电机驱动器。

### `d raw`

这进入“原始（raw）”PWM 模式。语法：

```
d raw <pwm_a> <pwm_b> <pwm_c>
```

其中 pwm 值介于 0.0 到 1.0 之间。因此，进入空闲状态的命令为：`d raw 0.5 0.5 0.5`。

### `d pwm`

这进入电压-FOC 模式。语法：

```
d pwm <phase> <magnitude> [<phase_rate>]
```

其中 phase（相位）以弧度为单位，magnitude（幅值）以伏特为单位，可选的 phase rate（相位速率）以弧度每秒为单位。

### `d dq`

这进入电流控制 FOC 模式。语法：

```
d dq <d_A> <q_A>
```

### `d pos`

这进入位置控制 FOC 模式。语法：

```
d pos <pos> <vel> <max_torque> [options...]
```

每个可选元素由一个前缀字符加一个值组成。允许的选项有：

- `p` - kp 比例：在此命令持续期间，已配置的 kp 值会乘以该常量
- `d` - kd 比例：在此命令持续期间，已配置的 kd 值会乘以该常量
- `i` - ki 积分限幅比例：在此命令持续期间，已配置的 ilimit 值会乘以该常量
- `s` - 停止位置：当给定了非零速度时，当控制位置达到该值时运动停止。
- `f` - 前馈转矩，单位为 Nm
- `t` - 超时（timeout）：如果在这么多秒内未收到另一个命令，则进入超时模式。
- `v` - 速度限制：在此命令持续期间，给定值将覆盖全局速度限制。
- `a` - 加速度限制：在此命令持续期间，给定值将覆盖全局加速度限制。
- `o` - 固定电压覆盖：在生效期间，将控制视为启用了具有给定电压的 `fixed_voltage_mode`
- `c` - 固定电流覆盖：在生效期间，将控制视为 `fixed_voltage_mode`，但改为命令一个固定电流。
- `b` - 如果非零，则忽略所有 `servopos` 位置界限

位置、速度、最大转矩以及所有可选字段与上文所述的寄存器协议具有相同的语义。

### `d tmt`

进入超时模式。该模式命令零速度，只能通过已停止状态退出。

```
d tmt <pos> <vel> <max_torque> [options...]
```

可用选项与 `d pos` 相同。

### `d zero`

进入零速度状态。无论位置如何，都命令零速度。

```
d zero <pos> <vel> <max_torque> [options...]
```

可用选项与 `d pos` 相同。`pos` 和 `vel` 会被忽略。

### `d within`

进入“保持在范围内（stay within）”状态。当位置包含在给定界限内时，仅施加前馈转矩。否则，使用位置模式控制器将位置保持在被违反的边界处。

```
d within <lowbound> <highbound> <max_torque> [options...]
```

各字段与上文所述的寄存器协议具有相同的语义。选项与 `d pos` 大致相同。不支持的选项包括：

 * `s` - 停止位置
 * `o` - 固定电压覆盖
 * `c` - 固定电流覆盖

### `d brake`

进入“制动（brake）”状态。在此模式下，所有电机相都短接到地，产生被动的“制动”作用。

### `d nearest`

将当前位置更新为与给定输出位置一致的最接近的位置。

```
d rezero <position>
```

### `d exact`

将当前位置更新为完全等于给定值。

```
d exact <position>
```

### `d req-reindex`

将归位（homing）状态重置为基于相对（relative），要求重新运行任何归位过程。

```
d req-reindex
```

### `d recapture`

当处于位置模式时，将控制位置和速度重置为当前感测到的值。

```
d recapture
```

### `d cfg-set-output`

按需修改配置，使当前观测到的位置等于给定值。注意，该配置不会写入持久存储。

```
d cfg-set-output <position>
```

### `d cal`

仅供 moteus_tool 内部使用：进入编码器校准模式。启动此命令后，控制器将在电压-FOC 模式下旋转电机，直到编码器覆盖完整一圈，然后反向重复该过程。在此过程中，当前命令相位和编码器值会周期性地输出到控制台。

语法：

```
d cal <magnitude> [options...]
```

每个可选元素由一个前缀字符加一个值组成。允许的选项有：

* `s` - 校准速度，单位为每秒电转数

注意：此命令仅供 moteus_tool 内部使用。它只执行校准过程的一部分，并且不会向持久存储保存任何内容。


### `d flash`

进入引导加载程序（bootloader）。

注意：这仅用于内部用途。想要刷写新固件的用户应使用 `moteus_tool`。

## `aux[12]` - Aux 端口操作

所有命令在 `aux1` 和 `aux2` 上都受支持。

### `aux1 out` - 设置 GPIO 输出值

```
aux1 out <data>
```

'data' 是一个单独的十进制整数。只有与配置为数字输出的引脚相关联的位才会被使用，其余位被忽略。

### `aux1 pwm` - 设置 PWM 输出值

```
aux1 pwm <pin> <value>
```

'pin' 是 0 到 4 之间的数字，给出要设置的引脚。

'value' 是介于 0.0 到 1.0 之间的浮点值，给出该引脚的输出占空比。

### `aux1 ic-cmd` - 发起一个 iC-PZ 命令

```
aux1 ic-cmd <HEXBYTE>
```

进入模拟模式校准的示例。

```
aux1 ic-cmd B0
```

### `aux1 ic-wr` - 写 iC-PZ 寄存器

```
aux1 ic-wr <reg> <data>
```

寄存器是一个十六进制字节，data 是一个或多个十六进制字节。

切换到存储器第 0 页的示例。

```
aux1 ic-wr 40 00
```

### `aux1 ic-rd` - 读 iC-PZ 寄存器

```
aux1 ic-rd <reg> <length>
```

寄存器是一个十六进制字节，length 是一个十进制值，表示要读取的字节数。

读取温度数据的示例：

```
aux1 ic-rd 4e 2
```

### `aux1 ic-extra` - 选择备用周期数据

```
aux1 ic-extra <fields>
```

"fields" 是一个十进制位掩码，表示要以 1000Hz 读取并在诊断流中显示的项。

bit 0 - 诊断命令结果
bit 1 - AI_PHASES 寄存器的内容

仅启用 AI_PHASES 寄存器的示例：

```
aux1 ic-extra 2
```

## `tel` - 遥测（telemetry）

### `tel get`

检索给定通道的内容。文本或二进制模式通过 `tel fmt` 或 `tel text` 命令确定，默认为二进制模式。

```
tel get <channel>
```

### `tel list`

列出所有可用的遥测通道。

### `tel schema`

报告与给定通道关联的二进制模式（schema）。

```
tel schema <channel>
```

该模式的报告方式如下：

```
emit <channel>\r\n
<LE uint32 size><data>
```

### `tel rate`

控制给定通道发出的速率。

```
tel rate <channel> <rate_ms>
```

### `tel fmt`

选择给定通道是以二进制还是文本形式发出。

```
tel fmt <channel> <format>
```

`format` 是一个整数，非零表示数据应以文本形式发出。

### `tel stop`

停止发出所有周期遥测数据。

### `tel text`

将所有通道切换到文本模式。

## `conf` - 配置

注意：任何更改参数的命令，例如 `conf set`、`conf load` 或 `conf default`，如果在 `tview` 中手动执行，不会自动更新 UI。必须重新启动 `tview` 才能在 UI 中显示新参数。

### `conf enumerate`

打印所有可配置参数的当前值。

### `conf get`

获取单个可配置参数的值。

```
conf get <item>
```

### `conf set`

在 RAM 中设置单个可配置参数的值。

```
conf set <item> <value>
```

### `conf load`

从持久存储加载所有可配置值。这将覆盖它们在 RAM 中的当前值。

### `conf write`

将所有可配置参数的当前值从 RAM 写入持久存储。

### `conf default`

将所有可配置参数的 RAM 值更新为其固件默认值。

注意：这仅更新 RAM，不更新持久存储。需要执行 `conf write` 才能将这些值保存到持久存储。
