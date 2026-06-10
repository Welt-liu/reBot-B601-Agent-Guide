# Write Motor ID

> **Note: This Agent Skill is a beta version and for reference only. If you have any questions or encounter issues, please contact Seeed for support.**
> **Reminder: Ensure motors are correctly connected and powered on before executing this Skill. Observe electrical safety precautions.**

## Prerequisites

The `/setup-env` Skill has been completed, with motorbridge and dependencies installed.

Activate the virtual environment before running commands:

```bash
conda activate rebot      # or the environment name you actually created, e.g. lerobot
```

> Motors will not respond if they are not powered on. If the scan fails, prompt the user to verify power status.

## Identify Device Type

Before proceeding, identify which protocol the motors use:

| Type | Connection | Description |
|------|------------|-------------|
| robstride | USB-CAN (PCAN-USB) | Communicates via socketcan `can0`, only modifies device_id, feedback_id remains unchanged |
| damiao | USB serial | Communicates via `/dev/ttyACM0`, requires modifying both ESC_ID and MST_ID |

## Damiao vs RobStride ID Writing Differences

| | Damiao | RobStride |
|---|---|---|
| Device ID register | `ESC_ID` (rid=8) | `device_id` |
| Feedback ID register | `MST_ID` (rid=7) | `host_id` (not modified) |
| Default device ID | `0x01` | `127` (0x7F) |
| Default feedback ID | `0x11` (= motor_id + 0x10) | `0xFD` (fixed value) |
| Modify feedback ID when changing ID | **Yes**, `feedback_id = motor_id + 0x10` | **No**, only device_id is changed |
| Parameters | `--motor-id` / `--new-motor-id` + `--feedback-id` / `--new-feedback-id` | `--motor-id` / `--new-motor-id` |

> Damiao motors have a fixed offset between `feedback_id` and `motor_id`: `feedback_id = motor_id + 0x10`.
> For example: motor_id=0x01 → feedback_id=0x11, motor_id=0x05 → feedback_id=0x15.

---

## Step 1: Device Initialization

### robstride (USB-CAN)

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

### damiao (USB serial)

1. Confirm the serial device exists:

```bash
ls -l /dev/ttyACM0
```

2. Check permissions. Output should show `crw-rw----` with group `dialout`. Confirm the current user is in that group:

```bash
groups
```

If the output does not include `dialout`, run:

```bash
sudo usermod -aG dialout $USER
```

> Re-login (or reboot) is required after adding to the group. After re-login, verify `groups` output includes `dialout`, otherwise serial port access will fail.

---

## Step 2: Check Existing Motors

Run the appropriate command for your device type:

**robstride:**
```bash
motorbridge-cli scan --vendor robstride --channel can0 --start-id 1 --end-id 8
```

**damiao:**
```bash
motorbridge-cli scan \
    --vendor damiao \
    --transport dm-serial \
    --serial-port /dev/ttyACM0 \
    --serial-baud 921600 \
    --start-id 1 \
    --end-id 32
```

- **robstride**: If motors 1–7 are detected, inform the user that the robotic arm may already be a finished product and ID modification is not needed. This Skill does not need to be executed.
- **damiao**: Determine whether modification is needed based on detected IDs.

---

## Step 3: Modify Motor ID

### Connect a New Motor

1. Ask the user to power off and unplug the cable from the current motor
2. Connect the cable to the next motor
3. After the user powers on, proceed to **Find Current Motor ID**

### Find Current Motor ID

Ensure the user has **only one motor connected** before proceeding.

**robstride:**
```bash
motorbridge-cli scan --vendor robstride --channel can0 --start-id 126 --end-id 127
```

**damiao:**
```bash
motorbridge-cli scan \
    --vendor damiao \
    --transport dm-serial \
    --serial-port /dev/ttyACM0 \
    --serial-baud 921600 \
    --start-id 1 \
    --end-id 1
```

- **robstride** hits **127**: Motor has not had its ID set. Proceed to **Modify Current Motor ID**.
- **damiao** hits **0x01**: Motor has not had its ID set. Proceed to **Modify Current Motor ID**.
- **Default ID not hit**: Expand the scan range (1–32). If a specific ID is found, the user has already set this motor. Inform the user of the detected ID and ask if they want to re-modify it.
- **Multiple motors detected**: Abnormal state — multiple motors are connected simultaneously. Ask the user to power off and unplug extra motors.

### Modify Current Motor ID

After confirming the default ID is detected, ask the user for the new motor ID and fill it into the CLI parameters:

**robstride:**
```bash
# Example: change ID 127 to 5
motorbridge-cli id-set --vendor robstride --motor-id 127 --new-motor-id 5
```

**damiao:**
```bash
# Example: change ESC_ID 0x05 to 1, MST_ID automatically changes from 0x15 to 0x11
motorbridge-cli id-set \
    --vendor damiao \
    --transport dm-serial \
    --serial-port /dev/ttyACM0 \
    --serial-baud 921600 \
    --motor-id 5 \
    --feedback-id 0x15 \
    --new-motor-id 1 \
    --new-feedback-id 0x11
```

On successful write, the CLI will output something like:

```
write rid=7 (MST_ID) <= 0x11
write rid=8 (ESC_ID) <= 0x1
store_parameters sent
verify rid=8 (ESC_ID): 0x1
verify rid=7 (MST_ID): 0x11
verify ok
```

> Damiao motors restart after ID modification. The command may end with a `get_register_u32 failed` timeout message. This indicates verification failed — immediately run a **scan** to confirm whether the ID was actually written. If not, re-modify.

---

## Step 4: Final Confirmation

Run the appropriate command for your device type:

**robstride:**
```bash
motorbridge-cli scan --vendor robstride --channel can0 --start-id 1 --end-id 8
```

**damiao:**
```bash
motorbridge-cli scan \
    --vendor damiao \
    --transport dm-serial \
    --serial-port /dev/ttyACM0 \
    --serial-baud 921600 \
    --start-id 1 \
    --end-id 32
```

On successful scan, the CLI will output something like:

```
[hit] vendor=damiao probe=0x01 esc_id=0x1 mst_id=0x11
[.. ] vendor=damiao probe=0x02 no reply
...
scan done: 1 motor(s) found
  probe=0x01 vendor=damiao esc_id=0x1 mst_id=0x11
```

Inform the user which motors were detected. Once the user confirms, this Skill is complete; otherwise, go back to re-modify.
