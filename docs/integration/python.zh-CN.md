# Python 客户端库

moteus 提供了一个 Python 库，可用于通过受支持的 CAN-FD 适配器命令和控制 moteus 控制器。

## 安装（Installation）

要安装，只需使用：

```
pip install moteus
```

或添加到您的 `requirements.txt` 文件中，或通过您项目所使用的任何机制。

## 基本用法（Basic Usage）

使用 Python 库有两种基本方法。第一种最简单，但如果总线利用率是设计需求，则它不能提供最佳的总线利用率。

首先，构造一个或多个控制器实例：

```python
import asyncio
import moteus

async def main():
    c1 = moteus.Controller(id=1)
    c2 = moteus.Controller(id=2)
    # ...

if __name__ == '__main__':
    asyncio.run(main())
```

然后，按固定的时间间隔，使用每个 API 的 `set_` 变体向每个设备发送命令。

```python
while True:
  c1_result = await c1.set_position(
      position=math.nan, velocity=1.0, accel_limit=0.5, query=True)
  c2_result = await c2.set_position(
      position=math.nan, velocity=0.5, accel_limit=0.25, query=True)

  c1_position = c1_result.values[moteus.Register.POSITION]
  c2_position = c2_result.values[moteus.Register.POSITION]

  print(c1_position, c2_position)

  await asyncio.sleep(0.01)
```


## 基于 `.cycle` 的用法

如果希望获得总线利用率或较高的更新率，可以使用一种替代 API 来最大化性能。在此 API 中，命令使用 `make_` 变体命令预先构造，然后以组的形式提交给库。

```python
import argparse
import asyncio
import moteus

async def main():
    parser = argparse.ArgumentParser()
    moteus.make_transport_args(parser)
    args = parser.parse_args()

    transport = moteus.get_singleton_transport(args)
    c1 = moteus.Controller(id=1, transport=transport)
    c2 = moteus.Controller(id=2, transport=transport)
    # ...

    while True:
        results = await transport.cycle([
           c1.make_position(position=math.nan, query=True)
           c2.make_position(position=math.nan, query=True),
           ])

        # Print the ID and position of all received responses.
        print(", ".join(
            f"{x.source} " +
            f"{x.values[moteus.Register.POSITION]}"
            for x in results))

        await asyncio.sleep(0.01)

if __name__ == '__main__':
    asyncio.run(main())
```

## 查询替代寄存器

默认情况下，只会查询有限的一组寄存器。完整的可用寄存器集合可在[寄存器参考文档](../protocol/registers.zh-CN.md)中找到。查询额外寄存器的最简单方法是向 `Controller` 构造函数传入一个替代的 `QueryResolution` 结构。

```python
qr = moteus.QueryResolution()
qr.power = moteus.F32
c = moteus.Controller(id=1, query_resolution=qr)
```

如果您想要的寄存器在 `QueryResolution` 结构中不可用，可以使用 `_extra` 方法来请求：

```python
qr = moteus.QueryResolution()
qr._extra = {
    moteus.Register.ENCODER_1_POSITION: moteus.F32,
    moteus.Register.ENCODER_1_VELOCITY: moteus.F32,
}
c = moteus.Controller(id=1, query_resolution=qr)
```

## API 参考

- [Python API 参考](../reference/python.zh-CN.md)
