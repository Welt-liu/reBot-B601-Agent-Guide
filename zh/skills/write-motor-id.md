# 写入电机 ID

> **注意：当前 Agent Skill 为测试版本，仅供参考。如有任何疑问或遇到问题，请联系 Seeed 获取支持。**
> **提醒：执行本 Skill 前请确保电机已正确连接且上电，操作过程中注意用电安全。**

## 使用前提

已完成 `/setup-env` Skill，安装 motorbridge 及依赖。

执行命令前需要激活虚拟环境：

```bash
conda activate rebot      # 或使用你实际创建的环境名，如 lerobot
```

> 扫描电机时如果没有上电是没有回应的，扫描失败时需要提示用户是否已上电。

## 确认设备类型

开始前先确认电机使用的协议类型：

| 类型 | 连接方式 | 说明 |
|------|----------|------|
| robstride（灵足） | USB-CAN（PCAN-USB） | 通过 socketcan `can0` 通信，仅修改 device_id，feedback_id 不变 |
| damiao（达妙） | USB 串口 | 通过 `/dev/ttyACM0` 通信，需同时修改 ESC_ID 和 MST_ID |

## 达妙 vs 灵足 ID 写入区别

| | 达妙（Damiao） | 灵足（RobStride） |
|---|---|---|
| 设备 ID 寄存器 | `ESC_ID`（rid=8） | `device_id` |
| 反馈 ID 寄存器 | `MST_ID`（rid=7） | `host_id`（不修改） |
| 默认设备 ID | `0x01` | `127`（0x7F） |
| 默认反馈 ID | `0x11`（= motor_id + 0x10） | `0xFD`（固定值） |
| 改 ID 时是否改反馈 ID | **是**，`feedback_id = motor_id + 0x10` | **否**，仅改 device_id |
| 参数 | `--motor-id` / `--new-motor-id` + `--feedback-id` / `--new-feedback-id` | `--motor-id` / `--new-motor-id` |

> 达妙电机的 `feedback_id` 与 `motor_id` 存在固定偏移：`feedback_id = motor_id + 0x10`。
> 例如 motor_id=0x01 → feedback_id=0x11，motor_id=0x05 → feedback_id=0x15。

---

## 第一步：设备初始化

### robstride（USB-CAN）

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

### damiao（USB 串口）

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

---

## 第二步：检查现有电机

根据设备类型执行对应命令：

**robstride：**
```bash
motorbridge-cli scan --vendor robstride --channel can0 --start-id 1 --end-id 8
```

**damiao：**
```bash
motorbridge-cli scan \
    --vendor damiao \
    --transport dm-serial \
    --serial-port /dev/ttyACM0 \
    --serial-baud 921600 \
    --start-id 1 \
    --end-id 32
```

- **robstride** 如果命中 1～7 号电机，提示用户当前机械臂可能为成品版本，不需要修改 ID，本 Skill 无需执行。
- **damiao** 根据命中的 ID 判断是否需要修改。

---

## 第三步：修改电机 ID

### 连接新电机

1. 要求用户断电后拔掉当前电机上的线
2. 将线连接到下一颗电机
3. 等待用户接上电源后，执行**查找当前电机 ID**

### 查找当前电机 ID

执行前要求用户**只能连接一颗电机**。

**robstride：**
```bash
motorbridge-cli scan --vendor robstride --channel can0 --start-id 126 --end-id 127
```

**damiao：**
```bash
motorbridge-cli scan \
    --vendor damiao \
    --transport dm-serial \
    --serial-port /dev/ttyACM0 \
    --serial-baud 921600 \
    --start-id 1 \
    --end-id 1
```

- **robstride** 命中 **127**：说明电机未设置过 ID，进入**修改当前电机 ID**步骤
- **damiao** 命中 **0x01**：说明电机未设置过 ID，进入**修改当前电机 ID**步骤
- **未命中默认 ID**：扩大扫描范围（1～32），若命中其中某个 ID，说明用户已设置过该电机。提示用户命中的 ID，询问是否需要重新修改
- **命中多个电机**：异常状态，说明同时连接了多个电机，要求用户在断电后拔掉多余电机。

### 修改当前电机 ID

确认命中默认 ID 后，要求用户给出新的 motor ID，填入 CLI 参数中：

**robstride：**
```bash
# 示例：将 ID 127 改为 5
motorbridge-cli id-set --vendor robstride --motor-id 127 --new-motor-id 5
```

**damiao：**
```bash
# 示例：将 ESC_ID 0x05 改为 1，MST_ID 自动从 0x15 改为 0x11
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

写入成功后 CLI 会输出类似以下内容：

```
write rid=7 (MST_ID) <= 0x11
write rid=8 (ESC_ID) <= 0x1
store_parameters sent
verify rid=8 (ESC_ID): 0x1
verify rid=7 (MST_ID): 0x11
verify ok
```

> 达妙电机修改 ID 后会重启，命令末尾可能出现 `get_register_u32 failed` 超时提示。这表示验证失败，需要立即执行**扫描**确认 ID 是否实际写入成功，如未成功则重新修改。

---

## 第四步：最后确认

根据设备类型执行对应命令：

**robstride：**
```bash
motorbridge-cli scan --vendor robstride --channel can0 --start-id 1 --end-id 8
```

**damiao：**
```bash
motorbridge-cli scan \
    --vendor damiao \
    --transport dm-serial \
    --serial-port /dev/ttyACM0 \
    --serial-baud 921600 \
    --start-id 1 \
    --end-id 32
```

扫描成功后 CLI 会输出类似以下内容：

```
[hit] vendor=damiao probe=0x01 esc_id=0x1 mst_id=0x11
[.. ] vendor=damiao probe=0x02 no reply
...
scan done: 1 motor(s) found
  probe=0x01 vendor=damiao esc_id=0x1 mst_id=0x11
```

告知用户命中了哪些电机。用户确认无误后，本 Skill 执行完成；否则返回重新修改。
