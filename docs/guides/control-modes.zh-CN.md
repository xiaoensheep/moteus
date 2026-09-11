# 控制模式（Control Modes）

moteus 控制器提供灵活的控制模式，以适应不同的机器人应用。通过调整控制增益和命令参数，你可以针对各种使用场景优化控制器——从精确定位和速度控制到高带宽转矩应用。本指南涵盖常见控制场景的推荐配置。

## 理解位置模式

moteus 控制器的主要控制模式是一个集成的位置/速度控制器。该控制命令的语义有些非常规，目的是便于从更高层控制器发送幂等命令。每条命令包含以下参数：

- **位置（Position）**：期望的位置，以转数为单位
- **速度（Velocity）**：期望位置变化的速率，单位为转/秒
- **最大转矩（Maximum torque）**：控制时使用的转矩绝不超过此值
- **前馈转矩（Feedforward torque）**：在正常控制回路给出的转矩之外额外提供这么多转矩
- **停止位置（Stop position）**：若非特殊值，则绝不让期望位置偏离此目标
- **kp scale**：按此因子缩放比例常数
- **kd scale**：按此因子缩放微分常数
- **ilimit scale**：按此因子缩放积分限制
- **速度限制覆盖（Velocity limit override）**：若非特殊值，则覆盖已配置的速度限制，该限制约束达到目标位置的快慢
- **加速度限制覆盖（Acceleration limit override）**：若非特殊值，则覆盖已配置的加速度限制，该限制约束达到目标位置的快慢

此外，位置可以设置为“特殊值”（浮点和调试接口中为 NaN，整数编码中为最大负值）。在这种情况下，所选位置为“你当前所在的位置”。

## 恒定加速度轨迹

速度与加速度限制既可以全局配置，也可以按每条命令配置，这会使 moteus 在内部生成连续的加速度受限轨迹，以达到给定位置和速度。轨迹完成后，命令速度会无限期地持续下去。

=== "诊断协议"

    ```
    # Move to position 1 then stop.  Accelerate/decelerate at 2Hz/s
    # and use a maximum velocity of 0.5Hz.
    d pos 1 0 nan a2 v0.5
    ```

=== "Python"

    ```python
    await controller.set_position(
        position=1,
        velocity=0,
        accel_limit=2,
        velocity_limit=0.5,
    )
    ```

=== "C++"

    ```cpp
    mjbots::moteus::Controller::Options options;
    options.position_format.accel_limit = mjbots::moteus::kFloat;
    options.position_format.velocity_limit = mjbots::moteus::kFloat;

    mjbots::moteus::Controller controller(options);

    mjbots::moteus::PositionMode::Command cmd;
    cmd.position = 1.0;
    cmd.velocity = 0.0;
    cmd.accel_limit = 2.0;
    cmd_velocity_limit = 0.5;

    auto result = controller.SetPosition(cmd);
    ```

加速度和速度限制的默认值也可以在配置中设置。

* `servo.default_accel_limit`
* `servo.default_velocity_limit`


## 速度控制

要实现速度控制器，每条命令的“位置”都应设置为 NaN（或等效的整数编码）。建议将 `servo.max_position_slip` 配置为一个大于或等于 0 的有限值（[参考](../reference/configuration.zh-CN.md#servomax_position_slip)）。该值越大，能抑制的外部扰动就越多，但当外部扰动幅值减小时，控制器也会“追赶”到位。

**示例：**

=== "诊断协议"

    ```
    d pos nan 2 nan
    ```

=== "Python"

    ```python
    await controller.set_position(
        position=math.nan,  # NaN for velocity mode
        velocity=2.0,       # 2 revolutions/second
        query=True
    )
    ```

=== "C++"

    ```cpp
    mjbots::moteus::PositionMode::Command cmd;
    cmd.position = std::numeric_limits<double>::quiet_NaN();
    cmd.velocity = 2.0;  // 2 revolutions/second

    auto result = controller.SetPosition(cmd);
    ```


## 转矩控制

对于纯转矩控制应用，位置控制回路的 PID 增益必须设为 0。实现这一目标的一种方式是将 `kp_scale`、`kd_scale` 和 `ilimit_scale` 值都设为 0。

=== "诊断协议"

    ```
    # Command a torque of 0.1 Nm
    d pos nan 0 nan p0 d0 i0 f0.1
    ```

=== "Python"

    ```python
    await controller.set_position(
        position=math.nan,
        velocity=0,
        kp_scale=0.0,
        kd_scale=0.0,
        ilimit_scale=0.0,
        feedforward_torque=0.1,
    )
    ```

=== "C++"

    ```cpp
    mjbots::moteus::Controller::Options options;
    options.position_format.kp_scale = mjbots::moteus::kFloat;
    options.position_format.kd_scale = mjbots::moteus::kFloat;
    options.position_format.ilimit_scale = mjbots::moteus::kFloat;
    options.position_format.feedforward_torque = mjbots::moteus::kFloat;

    mjbots::moteus::Controller controller(options);

    mjbots::moteus::PositionMode::Command cmd;
    cmd.position = std::numeric_limits<double>::quiet_NaN();
    cmd.velocity = 0.0;
    cmd.kp_scale = 0.0;
    cmd.kd_scale = 0.0;
    cmd.ilimit_scale = 0.0;
    cmd.feedforward_torque = 0.1;

    auto result = controller.SetPosition(cmd);
    ```

**注意事项**：使用外部转矩控制的带宽将远低于使用 moteus 内置位置控制器的带宽（若达到最大 CAN-FD 更新率，带宽约低 15 倍）。因此，如果尽可能多地将期望的控制律以内置位置控制器的方式表达，系统通常会有更好的表现。

如果系统除了转矩控制之外什么都不做，则可以在配置中将 PID 增益设为 0。

* `servo.pid_position.kp`
* `servo.pid_position.kd`
* `servo.pid_position.ilimit`

## 加加速度受限轨迹

moteus 仅支持加速度受限的内部轨迹。为近似恒定加加速度轨迹，主处理器应发送一系列分段线性的恒定速度轨迹来近似期望的轨迹。做法是以中等至高频率发送至少包含位置和速度的命令，同时禁用内部速度和加速度限制。

## 低速或精确定位

对于极低速运行，或者希望获得精确定位性能的情况，建议在位置控制器中配置非零的 `ki` 和 `ilimit` 项（[参考](../reference/configuration.zh-CN.md#servopid_position)）。这将补偿齿槽转矩（代价是整体转矩带宽降低）。此外，为校准选择更高的带宽值也可能有益。
