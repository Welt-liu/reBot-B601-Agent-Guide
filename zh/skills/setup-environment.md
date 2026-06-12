# 环境初始化

> **注意：当前 Agent Skill 为测试版本，仅供参考。如有任何疑问或遇到问题，请联系 Seeed 获取支持。**

## 第零步：检查是否已安装

在开始之前，先检查 motorbridge 是否已安装：

```bash
motorbridge-cli --help 2>/dev/null || python -m motorbridge --help 2>/dev/null || echo "motorbridge not installed"
```

- 如果输出帮助信息，说明 **motorbridge 已安装**，可跳过第一、二步，直接进入后续 Skill
- 如果输出 `motorbridge not installed`，继续执行以下步骤

---

## 第一步：准备 Python 环境

### Linux / macOS / Jetson / 树莓派

安装 Miniforge：

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

创建并激活虚拟环境：

```bash
conda create -y -n rebot python=3.12
conda activate rebot
```

### Windows

Windows 用户可以选择以下任一方式：

**方式 A：使用 Miniforge（推荐）**

1. 从 [Miniforge 发布页](https://github.com/conda-forge/miniforge/releases) 下载 Windows 安装器并安装
2. 打开 Anaconda Prompt，执行：

```bash
conda create -y -n rebot python=3.12
conda activate rebot
```

**方式 B：使用已有 Python 安装**

如果已安装 Python 3.10+，可以直接使用。确认 Python 路径：

```bash
python --version
```

确认 Python Scripts 目录在 PATH 中，或在命令中使用完整路径：
`<Python安装目录>\Scripts\motorbridge-cli.exe`

---

## 第二步：安装 motorbridge

```bash
pip install motorbridge
```

> 不锁定版本号，使用最新稳定版。安装完成后验证：

```bash
motorbridge-cli --help
```

如果提示命令未找到，检查 Python Scripts 目录是否在 PATH 中。Windows 用户使用完整路径执行。

---

## 第三步：平台特定配置

根据操作系统和设备类型，完成以下配置：

### Linux — robstride（USB-CAN）

```bash
# 加载 PCAN-USB 驱动
sudo modprobe peak_usb
ip -br link

# 配置 can0
sudo ip link set can0 down 2>/dev/null
sudo ip link set can0 type can bitrate 1000000 restart-ms 100
sudo ip link set can0 up

# 确认状态
ip -br link show can0
```

### Windows — robstride（USB-CAN）

Windows 使用 PCAN-USB **不需要配置 can0 接口**。

1. 首次使用需前往 [PCAN-USB 官方页面](https://www.peak-system.com/products/hardware/external-pc-interfaces/pcan-usb/) 下载并安装驱动
2. 插入 PCAN-USB 转接板
3. 无需执行任何额外配置命令，直接使用 `motorbridge-cli` 即可

### Linux — damiao（USB 串口）

1. 确认串口设备存在：

```bash
ls -l /dev/ttyACM0
```

2. 检查权限。输出应为 `crw-rw----` 且组为 `dialout`，确认当前用户在该组：

```bash
groups
```

如果输出中不包含 `dialout`，执行：

```bash
sudo usermod -aG dialout $USER
```

> 加入组后需要**重新登录**（或重启）才生效。重新登录后再次确认 `groups` 输出包含 `dialout`，否则无法访问串口。

### Windows — damiao（USB 串口）

1. 插入达妙电机 USB 串口线
2. 在设备管理器中确认 COM 端口号
3. 使用 `--serial-port COMx` 指定对应端口（如 `COM3`）
