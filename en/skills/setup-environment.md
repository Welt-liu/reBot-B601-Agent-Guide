# Environment Setup

> **Note: This Agent Skill is a beta version and for reference only. If you have any questions or encounter issues, please contact Seeed for support.**

## Step 0: Check for Existing Installation

Before starting, check whether motorbridge is already installed:

```bash
motorbridge-cli --help 2>/dev/null || python -m motorbridge --help 2>/dev/null || echo "motorbridge not installed"
```

- If help text is displayed, motorbridge is **already installed** — skip to the next Skill
- If `motorbridge not installed` is shown, proceed with the steps below

---

## Step 1: Prepare Python Environment

### Linux / macOS / Jetson / Raspberry Pi

Install Miniforge:

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

Create and activate a virtual environment:

```bash
conda create -y -n rebot python=3.12
conda activate rebot
```

### Windows

Windows users can choose one of the following:

**Option A: Using Miniforge (Recommended)**

1. Download the Windows installer from the [Miniforge releases page](https://github.com/conda-forge/miniforge/releases) and install
2. Open Anaconda Prompt and run:

```bash
conda create -y -n rebot python=3.12
conda activate rebot
```

**Option B: Using Existing Python Installation**

If Python 3.10+ is already installed, you can use it directly. Verify:

```bash
python --version
```

Ensure the Python Scripts directory is in PATH, or use the full path in commands:
`<Python_install_dir>\Scripts\motorbridge-cli.exe`

---

## Step 2: Install motorbridge

```bash
pip install motorbridge
```

> Do not pin to a specific version — use the latest stable release. After installation, verify:

```bash
motorbridge-cli --help
```

If the command is not found, check whether the Python Scripts directory is in PATH. Windows users should use the full path.

---

## Step 3: Platform-Specific Configuration

Complete the following setup based on your operating system and device type:

### Linux — robstride (USB-CAN)

```bash
# Load PCAN-USB driver
sudo modprobe peak_usb
ip -br link

# Configure can0
sudo ip link set can0 down 2>/dev/null
sudo ip link set can0 type can bitrate 1000000 restart-ms 100
sudo ip link set can0 up

# Verify status
ip -br link show can0
```

### Windows — robstride (USB-CAN)

Windows uses the PCAN-USB driver directly — **no can0 configuration needed**.

1. On first use, download and install the driver from the [PCAN-USB official page](https://www.peak-system.com/products/hardware/external-pc-interfaces/pcan-usb/)
2. Plug in the PCAN-USB adapter
3. No additional configuration required — use `motorbridge-cli` directly

### Linux — damiao (USB serial)

1. Confirm the serial device exists: `ls -l /dev/ttyACM0`
2. Confirm the user is in the `dialout` group: `groups`
3. If not in the group: `sudo usermod -aG dialout $USER` (requires re-login to take effect)

### Windows — damiao (USB serial)

1. Confirm the COM port number in Device Manager
2. Use `--serial-port COMx` in subsequent commands (e.g. `COM3`)
