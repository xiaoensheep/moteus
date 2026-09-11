# 硬件规格

## CAD 与模型

| 名称      | 2D CAD    | 3D 模型   |
|-----------|-----------|------------|
| moteus-c1 |[2D CAD](https://github.com/mjbots/moteus/blob/0.1-20240430/hw/c1/r1.2/20240305-moteus-c1-r1_2.pdf) | [STEP](https://github.com/mjbots/moteus/blob/0.1-20240430/hw/c1/r1.2/20240305-moteus-c1-r1_2.step) |
| moteus-r4 |[2D CAD](https://github.com/mjbots/moteus/blob/main/hw/controller/r4.5/20210124-moteus-controller-r45-mechanical.pdf) | [STEP](https://github.com/mjbots/moteus/blob/main/hw/controller/r4.5/20210124-moteus-controller-r45-mechanical.step) |
| moteus-x1 | [2D CAD](https://drive.google.com/file/d/1R7wuc7vk1khD5ZvDWPy54Bx4PgHYiUCu/view) | [STEP](https://drive.google.com/file/d/19tJa4gzy0ZBWYYsaJA-Du0PgzoDWizjL/view?usp=sharing) |
| moteus-n1 | [2D CAD](https://drive.google.com/file/d/1Ic65vT8BSeTtqz6uqd8C7l68aOMy3bMz/view?usp=share_link) | [STEP](https://drive.google.com/file/d/1wxX5G6kX6M6YZGGSceiXTOUZycxRaSIE/view?usp=share_link) |

## 电气限制

| 名称 | 最小电压 | 标称电压 | 绝对最大电压 | 峰值输出 | 3V 辅助 |  5V 辅助 | 12V 辅助 |
|------|-------------|---------|-----------------|-------------|--------|---------|---------|
| moteus-c1 | 10V | 24V | 51V (<= 12S) | 20A | 50mA | 100mA | N/A |
| moteus-r4 | 10V | 24V | 44V (<= 10S) | 100A | 100mA | N/A | N/A |
| moteus-x1 | 10V | 24V | 54V (<= 12S) | 120A | 100mA | 200mA | 150mA |
| moteus-n1 | 10V | 24V | 54V (<= 12S) | 100A | 100mA | 200mA | N/A |

## 功率限制

每个 moteus 控制器的允许最大功率取决于输入电压和 PWM 开关频率。下表给出 `servo.pwm_rate_hz=30000` 时的最大允许功率。

| 名称       | 峰值功率   |                | 高输入功率 |
|------------|--------------|----------------|------------------|
| moteus-r4  | <= 30V 900W  | 线性降额 | >= 38V 400W      |
| moteus-c1  | <= 28V 250W  | 线性降额 | >= 41V 150W      |
| moteus-n1  | <= 36V 2000W | 线性降额 | >= 44V 1000W     |

对于其他 `servo.pwm_rate_hz` 值，允许的最大功率随 PWM 速率线性变化，因此在 15000 时最大功率是上表的一半，在 60000 时是其两倍。但请注意，在更高的 PWM 速率下，控制器的效率会显著下降。

当前功率限制报告在 `servo_stats.max_power_W` 中。控制器将尝试限制输出相电流，以在任一方向上（即施加功率或再生能量）保持在该报告的功率限制内。
