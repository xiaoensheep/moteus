# 参考文档

本页面已重新整理。请使用导航菜单或下方链接查找您需要的内容。

## 快速链接

### 原理与运行

<a name="theory-of-operation"></a>
**[运行原理](https://mjbots.github.io/moteus/reference/theory/)** - 控制器架构与控制律

<a name="usage-modes"></a>
**[使用模式 / 控制模式](https://mjbots.github.io/moteus/guides/control-modes/)** - 位置、速度与转矩控制模式

### 配置

<a name="initial-parameters"></a>
**[初始参数 / 配置](https://mjbots.github.io/moteus/guides/configuration/)** - 配置入门

<a name="encoder-configuration"></a>
**[编码器配置](https://mjbots.github.io/moteus/guides/encoder-overview/)** - 编码器设置与配置

<a name="auxiliary-port"></a>
**[辅助端口](https://mjbots.github.io/moteus/reference/encoders/#auxiliary-port)** - 辅助连接器配置与引脚定义

<a name="aux1--enc"></a>
**[AUX1/ENC 引脚](https://mjbots.github.io/moteus/reference/encoders/#r411-pins-aux1enc)** - r4.11 AUX1/ENC 连接器引脚定义

<a name="aux2--abs"></a>
**[AUX2/ABS 引脚](https://mjbots.github.io/moteus/reference/encoders/#r411-pins-aux2abs)** - r4.11 AUX2/ABS 连接器引脚定义

<a name="aux1-out---set-gpio-output-values"></a>
**[AUX GPIO 输出控制](https://mjbots.github.io/moteus/protocol/diagnostic/#aux1-out-set-gpio-output-values)** - 通过诊断命令设置 GPIO 输出

<a name="aux12uartmode"></a>
**[AUX UART 模式](https://mjbots.github.io/moteus/reference/encoders/#uart)** - 在辅助端口上配置 UART

<a name="pin-options"></a>
**[引脚能力](https://mjbots.github.io/moteus/reference/encoders/#io-pin-capabilities)** - 可用的引脚模式与配置

### 配置参数

<a name="idid"></a>
**[id.id](https://mjbots.github.io/moteus/reference/configuration/#idid)** - 设备 ID 配置

<a name="servodefault_timeout_s"></a>
**[servo.default_timeout_s](https://mjbots.github.io/moteus/reference/configuration/#servodefault_timeout_s)** - 命令超时配置

<a name="servoflux_brake_margin_voltage"></a>
**[servo.flux_brake_margin_voltage](https://mjbots.github.io/moteus/reference/configuration/#servoflux_brake_margin_voltage)** - 磁通制动电压阈值

<a name="servomax_power_w"></a>
**[servo.max_power_W](https://mjbots.github.io/moteus/reference/configuration/#servomax_power_w)** - 最大功率限制

<a name="servopid_position"></a>
**[servo.pid_position](https://mjbots.github.io/moteus/reference/configuration/#servopid_position)** - 位置 PID 控制器参数

<a name="servoposposition_min"></a>
**[servopos.position_min](https://mjbots.github.io/moteus/reference/configuration/#servoposposition_min)** - 最小位置限制

<a name="motor_positionoutputoffsetsign"></a>
**[motor_position.output.offset/sign](https://mjbots.github.io/moteus/reference/configuration/#motor_positionoutputoffsetsign)** - 输出编码器偏移与符号

<a name="motor_positionrotor_to_output_ratio"></a>
**[motor_position.rotor_to_output_ratio](https://mjbots.github.io/moteus/reference/configuration/#motor_positionrotor_to_output_ratio)** - 减速比

<a name="conf-write"></a>
**[配置命令](https://mjbots.github.io/moteus/protocol/diagnostic/#conf-configuration)** - 通过控制台读取和写入配置

### 寄存器参考

<a name="0x000---mode"></a>
**[0x000 - 模式](https://mjbots.github.io/moteus/protocol/registers/#0x000-mode)** - 运行模式寄存器

<a name="0x00f---fault-code"></a>
**[0x00f - 故障代码](https://mjbots.github.io/moteus/protocol/registers/#0x00f-fault-code)** - 故障状态寄存器

<a name="0x014--0x15--0x16---voltage-phase-a--b--c"></a>
**[0x014/0x15/0x16 - 电压相 A/B/C](https://mjbots.github.io/moteus/protocol/registers/#0x014-0x15-0x16-voltage-phase-a-b-c)** - 相电压控制寄存器

<a name="0x020---position-command"></a>
**[0x020 - 位置指令](https://mjbots.github.io/moteus/protocol/registers/#0x020-position-command)** - 位置设定值寄存器

<a name="0x021---velocity-command"></a>
**[0x021 - 速度指令](https://mjbots.github.io/moteus/protocol/registers/#0x021-velocity-command)** - 速度设定值寄存器

<a name="0x023---kp-scale"></a>
**[0x023 - Kp 缩放](https://mjbots.github.io/moteus/protocol/registers/#0x023-kp-scale)** - 比例增益缩放

<a name="0x025---maximum-torque"></a>
**[0x025 - 最大转矩](https://mjbots.github.io/moteus/protocol/registers/#0x025-maximum-torque)** - 转矩限制寄存器

<a name="0x028---velocity-limit"></a>
**[0x028 - 速度限制](https://mjbots.github.io/moteus/protocol/registers/#0x028-velocity-limit)** - 轨迹速度限制

<a name="0x058---encoder-validity"></a>
**[0x058 - 编码器有效性](https://mjbots.github.io/moteus/protocol/registers/#0x058-encoder-validity)** - 编码器状态位域

<a name="0x05c---aux1-gpio-command"></a>
**[0x05c - Aux1 GPIO 指令](https://mjbots.github.io/moteus/protocol/registers/#0x05c-aux1-gpio-command)** - GPIO 输出控制寄存器

<a name="0x070---millisecond-counter"></a>
**[0x070 - 毫秒计数器](https://mjbots.github.io/moteus/protocol/registers/#0x070-millisecond-counter)** - 系统时间戳寄存器

<a name="0x131---set-output-exact"></a>
**[0x131 - 设置输出精确值](https://mjbots.github.io/moteus/protocol/registers/#0x131-set-output-exact)** - 强制绝对位置寄存器

<a name="0x150---0x153---uuid"></a>
**[0x150-0x153 - UUID](https://mjbots.github.io/moteus/protocol/registers/#0x150-0x153-uuid)** - 设备 UUID 寄存器

<a name="0x154---0x157---uuid-mask"></a>
**[0x154-0x157 - UUID 掩码](https://mjbots.github.io/moteus/protocol/registers/#0x154-0x157-uuid-mask)** - UUID 过滤寄存器

<a name="a2a-mappings"></a>
**[寄存器映射](https://mjbots.github.io/moteus/protocol/registers/#mappings)** - 数据类型编码与缩放

<a name="a2b-registers"></a>
**[寄存器列表](https://mjbots.github.io/moteus/protocol/registers/#registers)** - 完整的寄存器文档

### CAN 协议

<a name="f-can-fd-communication"></a>
<a name="a1-can-format"></a>
**[CAN 格式](https://mjbots.github.io/moteus/protocol/can/#can-format)** - CAN-FD 帧结构

<a name="a1a-write-registers"></a>
**[写入寄存器](https://mjbots.github.io/moteus/protocol/can/#write-registers)** - 寄存器写入子帧格式

<a name="a3-example"></a>
**[CAN 协议示例](https://mjbots.github.io/moteus/protocol/can/#example)** - CAN 帧拆解示例

<a name="sending-multiple-commands-at-once"></a>
**[一次发送多条指令](https://mjbots.github.io/moteus/reference/client-tools/#sending-multiple-commands-at-once)** - 在 tview 中批量发送指令

<a name="communicating-with-a-specific-device"></a>
**[与指定设备通信](https://mjbots.github.io/moteus/reference/client-tools/#communicating-with-a-specific-device)** - 通过 ID 定位设备

<a name="bit-timings"></a>
**[CAN 位时序](https://mjbots.github.io/moteus/platforms/socketcan/#bit-timings)** - socketcan 波特率配置

<a name="40mhz-clock-systems"></a>
<a name="80-mhz-clock-systems"></a>
**[特定时钟的位时序](https://mjbots.github.io/moteus/platforms/socketcan/#bit-timings)** - 40MHz 与 80MHz 配置

### 诊断命令

<a name="b-diagnostic-command-set"></a>
**[诊断协议](https://mjbots.github.io/moteus/protocol/diagnostic/)** - 控制台命令参考

<a name="d-pwm"></a>
**[d pwm](https://mjbots.github.io/moteus/protocol/diagnostic/#d-pwm)** - 原始 PWM 控制命令

<a name="d-nearest"></a>
**[d nearest](https://mjbots.github.io/moteus/protocol/diagnostic/#d-nearest)** - 设置输出最近值命令

<a name="d-cfg-set-output"></a>
**[d cfg-set-output](https://mjbots.github.io/moteus/protocol/diagnostic/#d-cfg-set-output)** - 配置输出编码器

### 硬件参考

<a name="electrical--pinout"></a>
<a name="pinout"></a>
**[引脚定义](https://mjbots.github.io/moteus/reference/pinouts/)** - 所有电路板型号的连接器引脚定义

<a name="jst-zh-6-swd"></a>
**[调试端口 (JST ZH-6)](https://mjbots.github.io/moteus/reference/pinouts/)** - SWD 调试连接器

<a name="moteus-n1c1---aux2---jst-gh-7"></a>
**[n1/c1 AUX2 (JST GH-7)](https://mjbots.github.io/moteus/reference/pinouts/)** - n1 与 c1 辅助端口连接器

<a name="moteus-r4---abs---jst-zh-4"></a>
**[r4 ABS (JST ZH-4)](https://mjbots.github.io/moteus/reference/pinouts/)** - r4 绝对式编码器连接器

<a name="moteus-r4---pico-spox-6-enc"></a>
**[r4 ENC (Pico-SPOX-6)](https://mjbots.github.io/moteus/reference/pinouts/)** - r4 增量式编码器连接器

**[硬件规格](https://mjbots.github.io/moteus/reference/hardware/)** - 电气额定值与机械信息

**[应用限制](https://mjbots.github.io/moteus/reference/limits/)** - 位置、速度与性能约束

### 校准与整定

<a name="calibration"></a>
**[校准](https://mjbots.github.io/moteus/guides/calibration/)** - 电机与编码器校准流程

<a name="pid-tuning"></a>
**[PID 整定](https://mjbots.github.io/moteus/guides/pid-tuning/)** - 控制器整定方法

### 固件与工具

<a name="building-firmware"></a>
<a name="flashing-over-can"></a>
<a name="from-the-debug-port"></a>
**[烧录与编译固件](https://mjbots.github.io/moteus/reference/firmware/)** - 固件编译与安装

<a name="tview-usage"></a>
**[tview 用法](https://mjbots.github.io/moteus/reference/client-tools/#tview-usage)** - 命令行工具文档

### 部署

<a name="deployment-considerations"></a>
<a name="design-considerations-for-regenerative-braking"></a>
<a name="regenerative-braking-safety"></a>
<a name="phase-wire-soldering"></a>
<a name="power-cable-construction"></a>
<a name="power-connectorization"></a>
<a name="long-daisy-chains"></a>
**[部署注意事项](https://mjbots.github.io/moteus/guides/electrical-setup/)** - 生产设计指南与安全注意事项

### 附加主题

<a name="hall-sensor"></a>
**[霍尔效应传感器](https://mjbots.github.io/moteus/reference/encoders/#hall-sensor)** - 使用带霍尔效应传感器的电机或配置霍尔传感器引脚

<a name="position"></a>
**[位置模式](https://mjbots.github.io/moteus/guides/control-modes/#understanding-position-mode)** - 位置控制详情

<a name="torque-control"></a>
**[转矩控制](https://mjbots.github.io/moteus/guides/control-modes/#torque-control)** - 直接转矩控制模式
