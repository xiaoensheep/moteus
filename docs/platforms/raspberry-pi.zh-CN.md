# Raspberry Pi 安装 #

非 GUI 的 moteus 库在 Raspberry Pi 上可以开箱即用：

```
python -m venv --system-site-packages moteus-venv
source moteus-venv/bin/activate
pip install moteus
```

然而，pypi 和 piwheels 都没有可用的 pyside2 库，而它是 `moteus_gui`（以及因此 tview）所需要的。幸运的是，Raspberry Pi OS 中已打包了该库。要使用那个版本，你可以执行以下操作：

```
sudo apt install python3-pyside2* python3-serial python3-can python3-matplotlib python3-qtconsole

source moteus-venv/bin/activate
pip install asyncqt importlib_metadata pyelftools
pip install --no-deps moteus moteus_gui
```

# 运行 moteus_tool 和 tview #

这些工具针对 pi3hat 的特定选项记录在 [https://github.com/mjbots/pi3hat/blob/master/docs/reference.md#usage-with-client-side-tools](https://github.com/mjbots/pi3hat/blob/master/docs/reference.md#usage-with-client-side-tools)

注意，要使用 pi3hat，所有 Python 脚本都必须以 root 身份运行。使用 sudo 是其中一种方式：

```
source moteus-venv/bin/activate
pip install moteus-pi3hat

sudo moteus-venv/bin/tview
```
