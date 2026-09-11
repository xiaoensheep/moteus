# Rust API 参考

moteus Rust 库分为两个 crate。完整的自动生成文档可在 docs.rs 上获取。

- [**moteus** 在 docs.rs 上](https://docs.rs/moteus/) — 高级控制器 API
- [**moteus-protocol** 在 docs.rs 上](https://docs.rs/moteus-protocol/) — 低级协议类型（兼容 `no_std`）

本页提供了关键类型及其角色的精选摘要。

## moteus crate

### 控制器

[`BlockingController`](https://docs.rs/moteus/latest/moteus/struct.BlockingController.html)
:   具有自动发现传输方式的同步控制器。
    适用于单线程使用的最简单 API。
    提供 `set_position()`、`set_stop()` 以及其他阻塞方法，这些方法发送命令并返回 `QueryResult`。

[`AsyncController`](https://docs.rs/moteus/latest/moteus/struct.AsyncController.html)
:   用于 `async`/`.await` 的异步控制器。
    支持包装的阻塞传输和真正的异步（tokio feature）传输。
    方法集与 `BlockingController` 相同，但所有方法都返回 future。

[`Controller`](https://docs.rs/moteus/latest/moteus/struct.Controller.html)
:   低级帧构建器。
    生成不带任何传输的 `Command` 值——用于自定义传输或嵌入式系统。
    提供 `make_position_command()`、`make_stop()`、`parse_query()` 等。

### 命令与结果

[`Command`](https://docs.rs/moteus/latest/moteus/struct.Command.html)
:   一条路由后的 moteus 消息，将目的地/源/前缀与序列化的多路复用协议数据组合在一起。
    由 `Controller::make_*()` 方法生成。
    通过 `into_frame()` 转换为线级 `CanFdFrame`。

[`command::PositionCommand`](https://docs.rs/moteus/latest/moteus/command/struct.PositionCommand.html)
:   位置模式参数：位置、速度、前馈转矩、转矩限制、PID 缩放、加速度/速度限制等。
    使用构建器模式——链式调用 `.position()`、`.velocity()`、`.maximum_torque()` 等。

[`command::StopCommand`](https://docs.rs/moteus/latest/moteus/command/struct.StopCommand.html)
:   清除故障并将控制器设置为停止模式的停止命令。

[`command::CurrentCommand`](https://docs.rs/moteus/latest/moteus/command/struct.CurrentCommand.html)
:   直接电流（d/q 轴）控制命令。

[`command::VFOCCommand`](https://docs.rs/moteus/latest/moteus/command/struct.VFOCCommand.html)
:   电压 FOC（磁场定向控制）命令。

[`command::StayWithinCommand`](https://docs.rs/moteus/latest/moteus/command/struct.StayWithinCommand.html)
:   保持在范围内（stay-within）边界命令——控制器将位置维持在规定限制内。

[`command::ZeroVelocityCommand`](https://docs.rs/moteus/latest/moteus/command/struct.ZeroVelocityCommand.html)
:   零速度模式命令——控制器主动保持零速度。

[`query::QueryFormat`](https://docs.rs/moteus/latest/moteus/query/struct.QueryFormat.html)
:   指定要查询哪些寄存器以及以什么分辨率。
    可自定义以请求默认值（模式、位置、速度、转矩）之外的额外遥测字段。

[`query::QueryResult`](https://docs.rs/moteus/latest/moteus/query/struct.QueryResult.html)
:   解析后的响应，包含模式、位置、速度、转矩、电压、温度、故障码以及其他遥测字段。

### 传输层

[`Transport`](https://docs.rs/moteus/latest/moteus/struct.Transport.html)
:   管理与 moteus 控制器之间的通信通道。
    提供 `cycle()`（批量发送/接收）、`write()`（即发即忘）、`read()`（接收非请求消息）和 `flush_read()`，与 Python API 匹配。

[`TransportOptions`](https://docs.rs/moteus/latest/moteus/struct.TransportOptions.html)
:   传输自动发现的配置。
    设置首选接口、超时和其他选项。

[`TransportOps`](https://docs.rs/moteus/latest/moteus/trait.TransportOps.html)
:   阻塞传输实现的 trait。
    实现此 trait 以创建自定义传输后端。

[`AsyncTransport`](https://docs.rs/moteus/latest/moteus/struct.AsyncTransport.html) *（需要 `tokio` feature）*
:   用于 tokio 真正非阻塞 I/O 的异步传输包装器。

[`AsyncTransportOptions`](https://docs.rs/moteus/latest/moteus/struct.AsyncTransportOptions.html) *（需要 `tokio` feature）*
:   异步传输创建的配置。

[`AsyncTransportOps`](https://docs.rs/moteus/latest/moteus/trait.AsyncTransportOps.html) *（需要 `tokio` feature）*
:   异步传输实现的 trait。

### 传输后端

[`transport::fdcanusb::Fdcanusb`](https://docs.rs/moteus/latest/moteus/transport/fdcanusb/struct.Fdcanusb.html)
:   fdcanusb 串行（CDC）传输。
    当 fdcanusb 设备通过 USB 连接时自动检测。

[`transport::socketcan::SocketCan`](https://docs.rs/moteus/latest/moteus/transport/socketcan/struct.SocketCan.html)
:   Linux SocketCAN 传输。
    使用内核 CAN 接口（例如 `can0`）。

[`transport::async_fdcanusb::AsyncFdcanusb`](https://docs.rs/moteus/latest/moteus/transport/async_fdcanusb/struct.AsyncFdcanusb.html) *（需要 `tokio` feature）*
:   使用 tokio-serial 的异步 fdcanusb 传输。

[`transport::async_socketcan::AsyncSocketCan`](https://docs.rs/moteus/latest/moteus/transport/async_socketcan/struct.AsyncSocketCan.html) *（需要 `tokio` feature）*
:   使用 tokio 的异步 SocketCAN 传输。

### 诊断与实用工具

[`DiagnosticStream`](https://docs.rs/moteus/latest/moteus/struct.DiagnosticStream.html)
:   阻塞诊断协议流，用于读取和写入配置值、固件信息以及其他诊断命令。

[`AsyncDiagnosticStream`](https://docs.rs/moteus/latest/moteus/struct.AsyncDiagnosticStream.html) *（需要 `tokio` feature）*
:   诊断流的异步变体。

[`move_to()`](https://docs.rs/moteus/latest/moteus/move_to/fn.move_to.html)
:   用于协调多伺服运动的自由函数。
    同时将多个伺服移动到目标位置，等待所有伺服到达。

[`async_move_to()`](https://docs.rs/moteus/latest/moteus/move_to/fn.async_move_to.html) *（需要 `tokio` feature）*
:   协调多伺服运动的异步变体。

[`Error`](https://docs.rs/moteus/latest/moteus/enum.Error.html)
:   错误枚举，包含变体：`Io`、`Timeout`、`NoResponse`、`Fault`、`InvalidResponse`、`NotConnected`、`DeviceNotFound`、`Protocol`。
    实现 `std::error::Error` 和 `From<std::io::Error>`。

### 工厂与发现

[`get_singleton_transport()`](https://docs.rs/moteus/latest/moteus/fn.get_singleton_transport.html)
:   返回共享的全局传输实例。
    在首次调用时使用自动发现创建传输。

[`create_default_transport()`](https://docs.rs/moteus/latest/moteus/fn.create_default_transport.html)
:   使用默认自动发现设置创建新传输。

[`register()`](https://docs.rs/moteus/latest/moteus/fn.register.html)
:   注册自定义传输工厂以用于自动发现。
    允许第三方传输后端参与自动传输选择。

[`DeviceAddress`](https://docs.rs/moteus/latest/moteus/struct.DeviceAddress.html)
:   通过 CAN ID、UUID 或两者来寻址设备。
    基于 UUID 的寻址可在总线上自动解析 ID。

## moteus-protocol crate

此 crate 兼容 `no_std`（带有可选的 `std` feature），并提供线协议类型：

[`CanFdFrame`](https://docs.rs/moteus-protocol/latest/moteus_protocol/struct.CanFdFrame.html)
:   原始 CAN-FD 帧，具有 `arbitration_id`、`data`、`size`、`brs` 和 `fdcan` 字段。
    不包含路由信息——路由由 moteus crate 中的 `Command` 处理。

[`Register`](https://docs.rs/moteus-protocol/latest/moteus_protocol/enum.Register.html)
:   所有 moteus 寄存器地址的枚举（模式、位置、速度、转矩、电压、温度、故障、编码器寄存器等）。

[`Resolution`](https://docs.rs/moteus-protocol/latest/moteus_protocol/enum.Resolution.html)
:   寄存器值分辨率：`Ignore`、`Int8`、`Int16`、`Int32`、`Float`。
    控制寄存器值的精度和线尺寸。

[`Mode`](https://docs.rs/moteus-protocol/latest/moteus_protocol/enum.Mode.html)
:   控制器运行模式枚举：`Stopped`、`Fault`、`Position`、`ZeroVelocity`、`StayWithin`、`Current` 等。

[`HomeState`](https://docs.rs/moteus-protocol/latest/moteus_protocol/enum.HomeState.html)
:   归位状态枚举：`Relative`、`Rotor`、`Output`。

[`Scaling`](https://docs.rs/moteus-protocol/latest/moteus_protocol/struct.Scaling.html)
:   寄存器缩放辅助工具，用于将定点值编码/解码为线格式或从线格式解码。

[`command::*`](https://docs.rs/moteus-protocol/latest/moteus_protocol/command/index.html)
:   所有命令类型的命令结构和序列化。
    每种命令类型都有一个对应的格式结构，控制发送哪些字段以及以什么分辨率。

[`query::*`](https://docs.rs/moteus-protocol/latest/moteus_protocol/query/index.html)
:   查询格式和带反序列化的结果类型。
    `QueryFormat` 控制请求哪些寄存器；`QueryResult` 保存解析后的响应。

[`calculate_arbitration_id()`](https://docs.rs/moteus-protocol/latest/moteus_protocol/fn.calculate_arbitration_id.html)
:   从源、目的地、前缀和回复标志计算 CAN 仲裁 ID。

[`parse_arbitration_id()`](https://docs.rs/moteus-protocol/latest/moteus_protocol/fn.parse_arbitration_id.html)
:   从 CAN 仲裁 ID 解析出源、目的地、前缀和回复标志。

## Feature 标志

`tokio`
:   使用 tokio 启用真正的异步传输。
    添加 `AsyncFdcanusb`、`AsyncSocketCan`、`AsyncTransport`、`AsyncDiagnosticStream` 和 `async_move_to()`。

`clap`
:   启用 `add_transport_args()` 以使用 clap 进行 CLI 参数解析。

## 集成指南

有关用法示例和教程，请参阅 [Rust 客户端集成指南](../integration/rust.zh-CN.md)。
