# Zero Calibration

> **Note: This Agent Skill is a beta version and for reference only. If you have any questions or encounter issues, please contact Seeed for support.**

## Prerequisites

- `/write-motor-id` has been completed, motor IDs are correctly set
- `can0` is configured and UP (configured in Step 1 of motor ID setup)

## Steps

### 1. Activate Environment and Start Gateway

```bash
conda activate rebot
motorbridge-gateway -- --bind 127.0.0.1:9002 --transport socketcan --channel can0
```

Keep the Gateway running — do not close this terminal.

### 2. Open Motorbridge Studio

Direct the user to: https://motorbridge.github.io/motorbridge-studio/

Complete the zero calibration via the web interface.
