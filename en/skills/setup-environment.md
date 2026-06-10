# Environment Setup

> **Note: This Agent Skill is a beta version and for reference only. If you have any questions or encounter issues, please contact Seeed for support.**

## Step 1: Install Miniforge

Supported platforms: Windows / Ubuntu / macOS / Jetson / Raspberry Pi

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

## Step 2: Create Virtual Environment

Create a Python 3.12 virtual environment:

```bash
conda create -y -n rebot python=3.12
```

Activate the environment. Each time you open a terminal to use reBot features, run:

```bash
conda activate rebot
```

## Step 3: Install motorbridge

```bash
pip install motorbridge==0.2.9
```
