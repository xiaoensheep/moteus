# 工作原理

moteus 控制器旨在使用磁场定向控制（Field Oriented Control，FOC）来驱动 3 相无刷电机（brushless motor）。它集成了一个磁编码器（magnetic encoder）用于检测转子位置，3 个半 H 桥用于向三个相切换功率，并在三个相上均具备电流检测能力。

主要控制模式（本文档其余部分标记为“位置”模式）是一个两级的级联控制器，两者都运行在开关频率（默认 30kHz）下。

最外层是一个可选的限制加速度和速度的轨迹规划器。在它内部是一个带可选前馈转矩的位置/速度集成 PID 控制器。该回路的输出是 FOC 控制器 Q 相的期望转矩/电流。

内层是一个电流模式 PI 控制器。其输出是 Q 相的期望电压值。然后使用磁编码器将 D/Q 相电压值映射到电机的 3 个相。

![Control Structure](control_structure.png)

更精确地说，“位置控制器”实现以下控制律：

```
acceleration = trajectory_follower(command_position, command_velocity)
control_velocity = command_velocity OR control_velocity + acceleration * dt OR 0.0
control_position = command_position OR control_position + control_velocity * dt
position_error = control_position - feedback_position
velocity_error = control_velocity - feedback_velocity
position_integrator = limit(position_integrator + ki * position_error * dt, ilimit)
torque = position_integrator +
         kp * kp_scale * position_error +
         kd * kd_scale * velocity_error +
         command_torque
```

而“电流控制器”实现以下控制律：

```
current_error = command_current - feedback_current
current_integrator = limit(current_integrator + ki * current_error, ilimit)
voltage = current_integrator + kp * current_error
```

由于位置模式回路的 PID 缩放可以通过 `kp_scale` 和 `kd_scale` 项逐周期调整，这使你可以让控制器以完整的位置/速度/转矩控制，或速度/转矩，或仅转矩，或任意组合在整个控制周期内无缝运行。
