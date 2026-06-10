# 环境初始化

> **注意：当前 Agent Skill 为测试版本，仅供参考。如有任何疑问或遇到问题，请联系 Seeed 获取支持。**

## 第一步：安装 Miniforge

支持平台：Windows / Ubuntu / macOS / Jetson / 树莓派

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

## 第二步：创建虚拟环境

创建 Python 3.12 虚拟环境：

```bash
conda create -y -n rebot python=3.12
```

激活环境。每次打开终端使用 reBot 相关功能时，都需要执行：

```bash
conda activate rebot
```

## 第三步：安装 motorbridge

```bash
pip install motorbridge==0.2.9
```
