# Rust 客户端库

moteus 提供了一个 Rust 客户端库，用于通过受支持的 CAN-FD 适配器命令和控制 moteus 控制器。该库由两个 crate 组成：

- **`moteus`** — 带有传输（transport）管理功能的高层控制器 API
- **`moteus-protocol`** — 低层的 `no_std` 协议编码/解码

## 安装（Installation）

将 moteus crate 添加到您的项目中：

```
cargo add moteus
```

或直接将其添加到您的 `Cargo.toml` 中：

```toml
[dependencies]
moteus = "0.5"
```

如需配合 tokio 的异步支持，请启用 `tokio` 特性（feature）：

```toml
[dependencies]
moteus = { version = "0.5", features = ["tokio"] }
```

如需 CLI 参数解析支持，请启用 `clap` 特性：

```toml
[dependencies]
moteus = { version = "0.5", features = ["clap"] }
```

## 基本用法 — BlockingController

使用 moteus 最简单的方式是 `BlockingController`，它提供同步的、阻塞式的通信：

```rust
use std::thread;
use std::time::Duration;
use moteus::{BlockingController, command::PositionCommand};

fn main() -> Result<(), moteus::Error> {
    // Auto-discovers transport on first use
    let mut ctrl = BlockingController::new(1);

    // Clear any faults
    ctrl.set_stop()?;

    // Send commands at regular intervals to prevent watchdog timeout
    loop {
        let result = ctrl.set_position(
            PositionCommand::new()
                .position(f32::NAN)
                .velocity(1.0)
                .accel_limit(0.5)
        )?;

        println!("Position: {} Velocity: {}", result.position, result.velocity);

        thread::sleep(Duration::from_millis(10));
    }
}
```

`BlockingController` 会在第一次需要它的操作时自动发现（discover）一个可用的传输（fdcanusb、socketcan）。

## 异步用法 — AsyncController

对于异步应用程序，请启用 `tokio` 特性并使用 `AsyncController`：

```rust
use moteus::AsyncController;
use moteus::command::PositionCommand;

#[tokio::main]
async fn main() -> Result<(), moteus::Error> {
    // Auto-discovers transport on first use
    let mut ctrl = AsyncController::new(1);

    ctrl.set_stop().await?;

    loop {
        let result = ctrl.set_position(
            PositionCommand::new()
                .position(f32::NAN)
                .velocity(0.5)
        ).await?;

        println!("Position: {}", result.position);

        tokio::time::sleep(std::time::Duration::from_millis(10)).await;
    }
}
```

## 低层用法 — Controller

`Controller` 类型提供帧构建功能，而不涉及任何传输。这对于自定义传输、嵌入式系统，或当您想要完全控制帧的发送和接收方式时非常有用。

```rust
use moteus::{Controller, command::PositionCommand};

let controller = Controller::new(1);

// Build a position command
let cmd = controller.make_position_command(
    &PositionCommand::new().position(0.5).velocity(1.0),
    true  // request query response
);

// Convert to wire frame
let frame = cmd.into_frame();

// Send frame via your own transport...
// frame.arbitration_id, frame.data, frame.size
```

将响应解析回查询（query）结果：

```rust
use moteus::query::QueryResult;

// After receiving a response frame from transport...
let result = controller.parse_query(&response_frame)?;
println!("Mode: {:?}, Position: {}", result.mode, result.position);
```

## 自定义查询解析（Query Resolution）

默认情况下，只会查询有限的一组寄存器。要请求额外的寄存器，请自定义 `QueryFormat`：

```rust
use moteus::{BlockingController, Resolution, Register};
use moteus::query::QueryFormat;

let mut query_format = QueryFormat::default();
query_format.power = Resolution::Float;

let mut ctrl = BlockingController::new(1);
ctrl.controller.query_format = query_format;
```

对于不在标准 `QueryFormat` 中的寄存器，请使用 `extra` 机制：

```rust
use moteus::{BlockingController, Resolution, Register};
use moteus::query::{QueryFormat, ExtraQuery};

let mut query_format = QueryFormat::default();
query_format.extra[0] = ExtraQuery {
    register: Register::Encoder1Position,
    resolution: Resolution::Float,
};

let mut ctrl = BlockingController::new(1);
ctrl.controller.query_format = query_format;
```

## 传输配置（Transport Configuration）

### 自动发现选项（Auto-Discovery Options）

使用 `TransportOptions` 配置传输自动检测：

```rust
use moteus::{BlockingController, TransportOptions};
use std::time::Duration;

let opts = TransportOptions::new()
    .socketcan_interfaces(vec!["can0"])
    .timeout(Duration::from_millis(200));

let mut ctrl = BlockingController::with_options(1, &opts);
```

### 显式传输（Explicit Transport）

您也可以直接指定一个传输：

```rust
use moteus::BlockingController;
use moteus::transport::socketcan::SocketCan;

let transport = SocketCan::new("can0")?;
let mut ctrl = BlockingController::new(1)
    .transport(transport);
```

## 诊断（Diagnostics）

`DiagnosticStream` 提供对 moteus 诊断协议的访问，用于读取和写入配置：

```rust
use moteus::DiagnosticStream;

let mut stream = DiagnosticStream::new(1)?;

// Read a configuration value
let response = stream.command("conf get servo.max_current_A")?;
println!("{}", response);

// Write a configuration value
stream.command("conf set servo.max_current_A 30.0")?;
```

## 多伺服协调运动（Multi-Servo Coordinated Moves）

`move_to` 函数提供协调的多伺服运动：

```rust
use moteus::move_to::{move_to, MoveToOptions, Setpoint};

let setpoints = vec![
    Setpoint { id: 1, position: 0.5, velocity: None, max_torque: None },
    Setpoint { id: 2, position: -0.3, velocity: None, max_torque: None },
];

let opts = MoveToOptions::default();
move_to(&setpoints, &opts)?;
```

## API 参考

- [Rust API 参考](../reference/rust.zh-CN.md) — 关键类型的精选摘要
- [moteus on docs.rs](https://docs.rs/moteus/) — 完整的自动生成 API 文档
- [moteus-protocol on docs.rs](https://docs.rs/moteus-protocol/) — 协议 crate 文档
