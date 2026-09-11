# 配置参数

本节描述了最终用户应用中最可能修改的可配置值。对所有值的更改会*立即*生效。这意味着，例如，在对控制参数进行大幅更改之前，明智的做法是先停止控制回路。也可能不是这样，这取决于你的目标。

## `id.id`

呈现在 CAN 总线上的伺服 ID。修改此值后，你需要立即调整与之通信的伺服 ID，才能继续通信或保存参数。

## `can.prefix`

一个 13 位整数，用作所有 CAN 通信 ID 的高 13 位。与 `id.id` 一样，此值立即生效，因此在更改之后，必须使用正确的前缀重新启动通信，才能执行诸如保存配置之类的操作。

## `servopos.position_min`

允许的最小控制位置值，以圈（rotation）为单位。如果为 NaN，则不施加任何限制。

## `servopos.position_max`

允许的最大控制位置值，以圈为单位。如果为 NaN，则不施加任何限制。

## `servo.pid_position`

这些配置位置模式的 PID 控制器。

* `kp/ki/kd` - PID 增益，单位为：
  * kp - 每圈 Nm
  * ki - 每圈 Nm/s
  * kd - 每圈/秒 Nm
* `iratelimit` - 积分项可以累积（wind up）的最大速率，单位为 N*m/s。<0 表示“无限制”
* `ilimit` - I 项的总最大值，单位为 Nm
* `max_desired_rate` - 如果非零，则命令位置的变化速率被限制为该频率（单位 Hz）。

注意，这些值是物理单位。因此 `kp` 值为 1，意味着输出端每 1 圈的误差将施加 1 Nm 的校正转矩（torque）。类似地，`kd` 值为 1，则每秒 1 圈的误差将产生 1 Nm 的校正转矩。再次注意，这些值是在输出端测量的，因此是在 `motor_position.rotor_to_output_ratio` 所隐含的位置、速度和转矩缩放*之后*。

## `servo.pid_dq_hz`

电流控制回路的带宽，单位为 Hz。电流控制器的 PI 增益会根据该带宽以及校准得到的电机电阻和电感自动计算。通常在校准期间由 `moteus_tool` 的 `--cal-bw-hz` 选项设置。

## `servo.max_current_desired_rate`

期望电流可以变化的最大速率，单位为 A/s。0 表示无限制。

## `servo.default_velocity_limit` / `servo.default_accel_limit`

施加于 moteus 内部生成的轨迹上的限制。如果任一为 `nan`，则该限制未设置。这些限制也可以在每条命令上单独覆盖。这些限制的语义如下：

- *两者都未设置（都为 nan）* 在这种情况下，位置和速度命令立即生效。控制位置将被初始化为命令位置，控制速度将被设置为命令速度。控制位置将以给定速度无限前进，直到达到命令停止位置。

- *任一已设置*：如果 x_c 是命令位置，v_c 是命令速度，t 是自收到命令以来的时间，则语义可描述为：“匹配由 x = x_c + v_c * t 定义的轨迹”。

  如果设置了加速度限制，则上述效果通过命令加速度为 [-accel_limit, 0, accel_limit] 之一来实现。如果未设置加速度限制，则速度会瞬时变化。

  如果设置了速度限制，则中间速度将满足 "-velocity_limit < velocity < +velocity_limit"。如果未设置，则速度可以增长到任意大小。

注意：这在内部被限制为不超过 `servo.max_velocity`。

## `servo.inertia_feedforward`

当设置为非零值，且当前加速度限制生效时，这将施加一个等于当前加速度乘以该可配置值的前馈（feedforward）转矩。这可用于改善短距离运动的响应瞬态，因为此时加速期不足以让正常的位置 PID 良好跟踪。

在理想情况下，你应将其设置为系统的转动惯量（moment of inertia），单位为 kg * m^2。

## `servo.voltage_mode_control`

当设置为非零值时，电流控制回路不闭合，所有以安培为单位的电流命令将改为视为电压模式命令（以伏特为单位），通过校准得到的相电阻（phase resistance）相关联。对于高绕组电阻的电机，默认的电流检测电阻（current sense resistor）太小，无法进行精确的电流检测，会导致显著的齿槽转矩（cogging torque）和电流检测噪声。如果无法更换电流检测电阻，可以使用此标志来实现平滑控制。缺点是在高速或面对外部扰动时，实际转矩将不再精确跟随所施加的转矩。

设置后，`servo.pid_dq_hz` 配置值将不再产生任何影响。

## `servo.fixed_voltage_mode`

如果非零，则不进行基于反馈的位置或电流控制。相反，根据当前命令位置和配置的电机极数（motor pole），向相端子施加固定电压。在此模式下，编码器（encoder）和电流检测电阻完全不用于控制。

这是一种类似于廉价无刷云台控制器的控制模式，它依赖于在电机绕组中持续消耗固定量的功率。

当此模式激活时，驱动器禁用时的上报位置和速度将为 0，启用时则精确等于控制位置。

在此模式下，各种降额（derating）限制不生效：

 * 温度转矩降额
 * 超出位置边界时的转矩降额
 * 最大电流限制
 * 命令的最大转矩

过温（over-temperature）仍会触发故障（fault）。

## `servo.fixed_voltage_control_V`

在固定电压控制模式下，施加到输出的电压。


## `servo.max_position_slip`

当为有限值时，这会强制限制控制位置与当前测量位置之间的差值，以圈为单位。它可用于在速度模式（velocity mode）下使用控制器时防止“追赶”（catching up）。

## `servo.max_velocity_slip`

当为有限值时，这会强制限制控制速度与当前测量速度之间的差值，以 Hz 为单位。它可用于确保在速度模式下当外部转矩超过最大值时仍遵守加速度限制。如果使用，通常 `servo.max_position_slip` 必须相对较小以避免不稳定。

## `servo.max_voltage`

如果输入电压达到此值，将触发故障并停止所有转矩。

## `servo.max_power_W`

如果设置，则允许的最大功率设为此值与出厂板功率配置（factory board power profile）中的较小者。

## `servo.override_board_max_power`

如果为 true，则即使 `servo.max_power_W` 大于出厂板功率配置，也将其用作功率限制。

## `servo.pwm_rate_hz`

要使用的 PWM 速率，默认为 30000。允许的值在 15000 到 60000 之间。较低的值提高效率，但会限制峰值功率并降低最大速度和控制带宽。

## `servo.temperature_margin`

当温度处于 `servo.fault_temperature` 的该摄氏度值范围内时，转矩开始被限制。

## `servo.fault_temperature`

如果温度达到此值，将触发故障并停止所有转矩。

## `servo.enable_motor_temperature`

如果为 true，则将通过板上的 TEMP 焊盘检测电机温度。

## `servo.motor_temperature_margin`

当电机温度处于 `servo.motor_fault_temperature` 的该摄氏度值范围内时，转矩开始被限制。

## `servo.motor_fault_temperature`

如果电机温度达到此值，将触发故障并停止所有转矩。

## `servo.fault_position_error`

如果为有限值，则当绝对位置控制误差超过此值时触发故障。位置控制误差是命令位置与测量位置之差，以圈为单位。如果设置为 `nan`（默认值），则不触发故障。

## `servo.fault_velocity_error`

如果为有限值，则当绝对速度控制误差超过此值时触发故障。速度控制误差是命令速度与测量速度之差，以每秒圈数为单位。如果设置为 `nan`（默认值），则不触发故障。

## `servo.max_regen_power_W`

当再生（regenerating）时，moteus 会在电机绕组中耗散能量，以将输入总线上的最大再生功率限制在该值以下。

`nan` 禁用此功能，此时只有磁通制动（flux braking）会导致能量在绕组中耗散。

`0` 表示不允许任何再生功率进入输入总线。

警告：绕组中可以耗散的功率量受到板的电流限制，以及电源电压和相绕组电阻的限制。一旦 moteus 达到可施加到相绕组的最大功率，进一步的功率将被导向输入总线，无论此设置的值如何。

## `servo.flux_brake_margin_voltage`

相对于当前配置的 `servo.max_voltage` 选择磁通制动点。`磁通制动点 = max_voltage - flux_brake_margin_voltage`。

当输入电压高于制动点时，控制器使电机充当电阻为 `servo.flux_brake_resistance_ohm` 的“虚拟电阻”。所有额外能量都被倾泻到电机的 D 相。如果输入直流母线无法接受足够能量，这可用于处理过量的再生能量。

## `servo.max_current_A`

相电流（phase current）的使用永远不会超过此值。可以降低它以限制控制器使用的总功率。增加到超过出厂配置值可能导致硬件损坏。

## `servo.max_velocity`

如果速度超过此阈值，输出功率将被限制。

## `servo.max_velocity_derate`

一旦速度达到 max_velocity 加上此值，允许的输出功率将降为 0。

## `servo.rotation_*`

这些值配置电机的高阶转矩模型。

* `servo.rotation_current_cutoff_A` 如果相电流小于此值，则使用 `motor.Kv` 所隐含的线性关系。

一旦超过该截止值，使用以下公式从相电流确定转矩：

```
torque = cutoff * tc + torque_scale * log2(1 + (I - cutoff) * current_scale)
```

其中 `tc` 是由 `motor.Kv` 导出的转矩常数。

此模型不会自动校准，需要手动确定和配置。

## `servo.default_timeout_s`

当通过 CAN 发送位置模式命令时，存在一个可选的看门狗超时（watchdog timeout）。如果命令没有以一定速率到达，控制器将锁存进入“位置超时”（position timeout）状态，需要停止命令才能恢复运行。此配置值控制控制器在收到命令后进入此状态之前等待的时间长度。如果设置为 `nan`，则控制器永远不会进入此超时状态。

可以通过 0x027 寄存器或诊断接口 `d pos` 命令的 `t` 可选标志在每条命令上覆盖它。

## `servo.timeout_max_torque_Nm`

处于“位置超时”模式时，控制器会阻尼输出。此参数控制此类阻尼可用的最大转矩。

## `servo.timeout_mode`

选择在位置超时模式下将发生的行为。允许的值是顶层模式的子集。

* 0 - “停止”（stopped） - 驱动器脱开
* 10 - “减速到 0 速度并保持位置”
* 12 - “零速度”（zero velocity）
* 15 - “制动”（brake）

对于模式 10，使用 `servo.default_velocity_limit` 和 `servo.default_accel_limit` 来控制减速到零速的减速曲线。使用默认 PID 增益。在此超时模式下，转矩的唯一限制是 `servo.max_current_A`。

## `servo.motor_thermistor_ohm`

任何已连接电机 NTC 热敏电阻在 25C 时的电阻，单位为欧姆。

## `servo.fw.enable`

如果为 true，则允许在基速以上运行。控制器将使用负的 D 轴电流来降低有效 Kv 值，从而在基速以上运行时导致功率耗散增加。

## `servo.fw.max_current_ratio`

在弱磁（field weakening）运行时，D 轴电流幅值被限制为不超过此值乘以 `servo.max_current_A`。

## `aux[12].pins.X.mode`

选择在给定引脚上使用的功能。

* 0 - NC - 未连接（或用于板载 SPI）
* 1 - SPI - 用于 CLK、MISO 或 MOSI 之一
* 2 - SPI CS - 用于 SPI CS
* 3 - UART
* 4 - 软件正交（Software quadrature）
* 5 - 硬件正交（Hardware quadrature）
* 6 - 霍尔（Hall）
* 7 - 索引（Index）
* 8 - 正弦（Sine）
* 9 - 余弦（Cosine）
* 10 - Step（未实现）
* 11 - Dir（未实现）
* 12 - RC PWM（未实现）
* 13 - I2C
* 14 - 数字输入
* 15 - 数字输出
* 16 - 模拟输入
* 17 - PWM 输出

## `aux[12].pins.X.pull`

在每个引脚上配置可选的上拉或下拉。并非所有上拉选项都会与每种模式一起使用。此外，moteus 4.5/8/11 上的 2 个 aux2 引脚带有硬安装的 2k 欧姆上拉电阻，无论这些设置如何。

* 0 - 无上拉或下拉
* 1 - 上拉
* 2 - 下拉
* 3 - 开漏（open drain）（未实现）

## `aux[12].i2c.i2c_hz`

I2C 总线运行频率。介于 50000 和 400000 之间。

## `aux[12].i2c.i2c_mode`

要使用的 I2C 模式。

## `aux[12].i2c.pullup`

在 I2C 专用引脚上配置 I2C 专用的 2.2k 上拉电阻。如果设置，且该板此辅助端口上不可配置 I2C 上拉，则会为该 aux 端口报告错误。

## `aux[12].i2c.devices.X.type`

预期使用的 I2C 设备。

* 0 - 禁用
* 1 - AS5048
* 2 - AS5600

## `aux[12].i2c.devices.X.address`

要使用的 I2C 地址。

## `aux[12].i2c.devices.X.poll_rate_us`

每隔多少微秒轮询设备以获取更多数据。必须不小于 100。

## `aux[12].spi.mode`

SPI 设备的类型。

* 0 - 板载 AS5047P（CPR == 16384）。仅对 aux1 有效。如果选择，则 CLK、MOSI 和 MISO 线必须为 NC 或选择为 SPI。
* 1 - 禁用。
* 2 - AS5047P（CPR == 16384）
* 3 - iC-PZ
* 4 - MA732（CPR == 65536）
* 5 - MA600（CPR == 65536）
* 8 - AMT22（CPR == 16384）
* 9 - RLS Orbis（CPR == 16384）

注意：iC-PZ 设备在使用前需要进行大量配置和校准。诊断模式命令提供了底层访问。

## `aux[12].spi.rate_hz`

SPI 总线运行频率。默认为 12000000。

## `aux[12].uart.mode`

UART 设备的类型。

* 0 - 禁用
* 1 - RLS AksIM-2
* 2 - Tunnel（隧道）
* 3 - 每个控制周期的调试信息（未记录）
* 4 - CUI AMT21x 系列 RS422
* 5 - fdcanusb 串行协议，参见 [UART](../integration/uart.zh-CN.md)
* 6 - “板默认” - 启动时解析为板和端口特定的默认值

选择隧道模式时，可以使用 CAN 诊断协议发送或接收数据。对于 aux1，使用诊断通道 2。对于 aux2，使用诊断通道 3。

此值的默认值因辅助端口和 moteus 板而异。

* moteus-r4: aux1/禁用(0), aux2/fdcanusb(5)
* moteus-c1: aux1/禁用(0), aux2/fdcanusb(5)
* moteus-n1: aux1/fdcanusb(5), aux2/禁用(0)
* moteus-x1: aux1/fdcanusb(5), aux2/禁用(0)

如果某个辅助端口没有正确配置 RX 和 TX 引脚，则 `uart.mode` 必须设置为禁用(0)。

## `aux[12].uart.baud_rate`

UART 使用的波特率。

## `aux[12].uart.poll_rate_us`

对于编码器模式，轮询编码器获取新位置信息的时间间隔。

## `aux[12].uart.cui_amt21_address`

选择要通信的 CUI AMT21 地址。默认为 0x54（十进制 84），这是 CUI AMT21 编码器默认配置的地址。

## `aux[12].quadrature.enabled`

如果要从该端口读取正交输入，则为 true/非零。

## `aux[12].quadrature.cpr`

正交输入的每圈计数数（CPR）。如果用作源，则此 CPR 必须与源中配置的一致。

## `aux[12].hall.enabled`

如果要从该端口读取霍尔效应传感器，则为 true/非零。

## `aux[12].hall.polarity`

与 3 个霍尔相进行异或的位掩码。

## `aux[12].index.enabled`

如果要从该端口读取索引输入，则为 true/非零。

## `aux[12].sine_cosine.enabled`

如果要从该端口读取正弦/余弦输入，则为 true/非零。

## `aux[12].sine_cosine.common`

用于正弦/余弦的共模电压。采样使用 12 位，因此 2048 将精确等于 0.5 * 3.3V。但是，为了获得最佳性能，最好根据通过诊断协议观察到的实际读数来校准此值。

## `aux[12].bissc.enabled`

如果要在此端口上连接 BiSS-C 传感器，则为 true/非零。BiSS-C 编码器需要两个引脚，其中一个既可用作 UART-TX 引脚也可用作 PWM 引脚。

## `aux[12].bissc.rate_hz`

用于 BiSS-C 通信的比特率。1,000,000（1Mbit）是支持的最大速率。

## `aux[12].bissc.data_bits`

BiSS-C 编码器上报的数据位数。

## `aux[12].bissc.crc_bits`

BiSS-C 编码器上报的 CRC 位数。

## `aux[12].bissc.poll_rate_us`

轮询 BiSS-C 编码器的最小周期。

## `aux[12].i2c_startup_delay_ms`

上电后（或重新配置后），在该辅助端口关联的 I2C 设备首次被使用之前的延时，以毫秒为单位。

## `aux[12].pwm_period_us`

该辅助端口上 PWM 输出使用的周期，以微秒为单位。

## `aux[12].rs422`

启用 RS422 收发器。这仅对 'aux1' 有效，并要求引脚 D 和 E（`aux1.pins.3` 和 `aux1.pins.4`）用于 UART 或 BiSS-C。

## `motor_position.sources.X.aux_number`

aux1 设备为 1，aux2 设备为 2。

## `motor_position.sources.X.type`

以下之一：

* 0 - 禁用
* 1 - SPI
* 2 - UART
* 3 - 正交（Quadrature）
* 4 - 霍尔（Hall）
* 5 - 索引（Index）
* 6 - 正弦/余弦（Sine/Cosine）
* 7 - I2C
* 8 - 无传感器（Sensorless）（未实现）

注意：“索引”源类型仅允许用于输出参考（output referenced）编码器，可作为进入“输出”归位状态（homed state）的替代方式。

## `motor_position.sources.X.i2c_device`

如果 `type` 为“7/I2C”，这是一个从 0 开始的索引，指定应使用该端口上的*哪个* I2C 设备。

## `motor_position.sources.X.incremental_index`

如果指定的辅助端口具有增量编码器（如正交编码器），可将其设置为 1 或 2，以使用“索引”引脚将数据参考到给定位置。它将允许该源为换相（commutation）提供 theta 读数。

## `motor_position.sources.X.cpr`

给定输入的 CPR。在某些情况下会自动设置，但在大多数情况下需要手动输入。

## `motor_position.sources.X.offset/sign`

要应用的整数偏移和反相。结果值为：`(raw + offset) * sign / cpr`。

## `motor_position.sources.X.reference`

* 0 - 此源相对于转子（rotor）
* 1 - 此源相对于输出

## `motor_position.sources.X.pll_filter_hz`

选择用于该源的低通滤波器（low-pass filter）的 3dB 截止频率。它通常应小于输入更新速率的 10 倍，如果用作换相或输出传感器，则应高于被控对象（plant）的机械带宽。在该范围内，可以在可闻噪声与性能之间进行权衡。

如果设置为 0，则不应用滤波器。在这种情况下，本身不测量速度的传感器将不会产生速度读数（大多数传感器都是如此）。

## `motor_position.commutation_source`

源列表中的从 0 开始的索引，选择用于换相的源。这意味着它应当能够准确测量转子与定子（stator）之间的关系。

不建议使用正交源进行换相，因为它们可能丢失计数，并且每次上电都需要额外的应用层归位支持。

## `motor_position.output.source`

源列表中的从 0 开始的索引，选择用于输出位置的源。该源的位置和速度将在“位置”模式下用于控制。

## `motor_position.output.offset/sign`

偏移是一个以输出圈数为单位的浮点值。结合 -1/1 的符号，可用于定位输出的 0 点并控制其旋转方向。

## `motor_position.output.reference_source`

如果非负，这是源列表中的从 0 开始的索引。所选源在开机时用于消除多圈场景或配置了减速器时输出位置的歧义。

## `motor_position.rotor_to_output_ratio`

转子每转一圈，输出转动的圈数。对于齿轮减速器（几乎所有配置），此值将小于 1。例如，4 倍齿轮减速应输入为 0.25。

## `motor_position.rotor_to_output_override`

如果你*非常*清楚自己在做什么，并且想要配置非减速比，可将其设置为 true/非零。否则，大于 1.0 的比将导致故障。仅当系统具有不是减速器、而是加速输出的齿轮箱时，才应使用此选项。
