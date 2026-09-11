# 引脚定义

## moteus-c1

<img src="../c1/moteus-c1-pinout-rendered.svg" width="100%" />

## moteus-r4

<img src="../r4/moteus-r4-pinout-rendered.svg" width="100%" />

## moteus-x1

<img src="../x1/moteus-x1-pinout-rendered.svg" width="100%" />

## moteus-n1

<img src="../n1/moteus-n1-pinout-rendered.svg" width="100%" />

## 附加信息

### CAN 终端匹配

CAN 连接应在总线两端用 120 欧姆电阻进行终端匹配。某些 mjbots 产品内置了终端匹配电阻，例如 pi3hat。mjcanfd-usb-1x 和 fdcanusb 具有软件可配置的终端匹配电阻，默认开启。moteus 控制器没有终端匹配电阻。对于非常短的走线，系统只在单侧终端匹配即可工作。但是，当走线长度超过 0.5 米时，你可能需要对两端都进行终端匹配。这可以通过将 120 欧姆电阻压接到 JST PH3 连接器中并连接到开放数据连接器来实现，或购买 [CAN 终端匹配器](https://mjbots.com/products/jst-ph3-can-fd-terminator)。

### 推荐的对接硬件

| 连接器 | 对接件 P/N | 端子 P/N | 预压接导线 |
|-----------|----------|--------------|------------------|
| JST PH-3  | [PHR-3](https://mjbots.com/products/phr-3)  | [SPH-002T-P0.5L](https://mjbots.com/products/jst-ph-terminal) | [10cm, 30cm, 50cm PH3](https://mjbots.com/products/jst-ph3-cable) |
| JST GH-6  | [GHR-06V-S](https://mjbots.com/products/jst-gh6-housing) | [SSHL-002T-P0.2](https://www.digikey.com/en/products/detail/jst-sales-america-inc/SSHL-002T-P0-2/807828) | [28 AWG 20cm](https://mjbots.com/products/jst-gh-wire) |
| JST GH-7  | [GHR-07V-S](https://mjbots.com/products/jst-gh7-housing) | [SSHL-002T-P0.2](https://www.digikey.com/en/products/detail/jst-sales-america-inc/SSHL-002T-P0-2/807828) | [28 AWG 20cm](https://mjbots.com/products/jst-gh-wire) |
| JST GH-8  | [GHR-08V-S](https://mjbots.com/products/jst-gh8-housing) | [SSHL-002T-P0.2](https://www.digikey.com/en/products/detail/jst-sales-america-inc/SSHL-002T-P0-2/807828) | [28 AWG 20cm](https://mjbots.com/products/jst-gh-wire) |
| JST ZH-4  | [ZHR-4](https://www.digikey.com/en/products/detail/jst-sales-america-inc/ZHR-4/608643) | [SZH-002T-P0.5](https://www.digikey.com/en/products/detail/jst-sales-america-inc/SZH-002T-P0-5/527363) | |
| JST ZH-6  | [ZHR-6](https://www.digikey.com/en/products/detail/jst-sales-america-inc/ZHR-6/527361) | [SZH-002T-P0.5](https://www.digikey.com/en/products/detail/jst-sales-america-inc/SZH-002T-P0-5/527363) | [stm32 programmer](https://mjbots.com/products/stm32-programmer) |
| AMass XT30 | [XT30U-F](https://mjbots.com/products/xt30u-f) | | |

### 推荐的廉价手动压接工具

- **JST PH**: [TU-190-08](https://www.amazon.com/TU-190-08-Terminals-Tool-Crimping-0-08-0-5sq-mm/dp/B01M25OLZY)
- **JST ZH**: [PEBA 1020M](https://www.amazon.com/PEBA-Ratcheting-Connectors-Crimping-terminals/dp/B0D7PKLBZT)
- **JST GH**: [PEBA 1020M](https://www.amazon.com/PEBA-Ratcheting-Connectors-Crimping-terminals/dp/B0D7PKLBZT)

### moteus-r4 - Pico-SPOX 6 ENC

moteus-r4 上的 ENC/AUX1 连接器应安装 Molex Pico-SPOX 6 连接器，部件号 PN 0874380643，或者作为替代，TE 5-1775444-6。
