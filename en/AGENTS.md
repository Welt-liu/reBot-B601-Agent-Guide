# B601-RS Robotic Arm User Guide

> This document applies only to the **B601-RS robotic arm**.
> Except for commands that require explicit user permission, all steps are executed by the agent with your consent. In non-exceptional cases, do not ask the user to handle things manually in the terminal.
> Do not use motorbridge-cli commands not mentioned in this document, as they may start the motors and cause unexpected behavior.

## Quick Start

Please confirm which form of robotic arm you have:

| Form | Description | Starting Step |
|------|-------------|---------------|
| Kit (unassembled) | Requires user assembly | Start from Step 1 |
| Assembled | Pre-assembled and ready to use | Jump to Step 3 |

---

## Step 1: Environment Initialization

Run the [Setup Environment Skill](https://github.com/Welt-liu/reBot-B601-RS-Skills/blob/main/skills/en/setup-environment.md) to complete the following:

- [ ] Miniforge installation
- [ ] motorbridge configuration
- [ ] Gateway installation

After completion, ensure the virtual environment is activated:

```bash
conda activate rebot
```

---

## Step 2: Write Motor IDs

Run the [Write Motor ID Skill](https://github.com/Welt-liu/reBot-B601-RS-Skills/blob/main/skills/en/write-motor-id.md)

This Skill will automatically:

1. **Identify the USB-CAN adapter** (PCAN-USB, configure `can0` at bitrate `1000000`)
2. **Set motor IDs one by one** (7 motors total, ID range 1–7)
   - Connect only one motor at a time
   - Scan to confirm the current motor ID (default: 127)
   - Modify to the user-specified target ID
3. **Final verification** — Scan to confirm all motor IDs (1–7) are correctly written

---

## Step 3: Zero Calibration

Run the `/write-motor-zero` Skill.

This Skill will:

1. **Start motorbridge-gateway** (bound to `127.0.0.1:9002`, using `can0`)
2. **Guide the user to open Motorbridge Studio** to complete zero calibration

---

## Skills Status

| Skill | File | Status |
|-------|------|--------|
| `/setup-env` | `skills/en/setup-environment.md` | Completed |
| `/write-motor-id` | `skills/en/write-motor-id.md` | Completed |
| `/write-motor-zero` | `skills/en/write-motor-zero.md` | Completed |
