# 零点校准

> **注意：当前 Agent Skill 为测试版本，仅供参考。如有任何疑问或遇到问题，请联系 Seeed 获取支持。**

## 使用前提

- 已完成 `/write-motor-id`，电机 ID 已正确写入
- `can0` 已配置且处于 UP 状态（第二步中已配置）

## 步骤

### 1. 激活环境并启动 Gateway

```bash
conda activate rebot
motorbridge-gateway -- --bind 127.0.0.1:9002 --transport socketcan --channel can0
```

Gateway 启动后保持运行，不要关闭该终端。

> **重要：Gateway 会独占 CAN 总线**。在 Gateway 运行期间，`motorbridge-cli scan` 和其他 CLI 命令将无法与电机通信，会返回 `0 motor(s) found`。
> 如需在 Gateway 运行期间扫描电机 ID，必须先停止 Gateway：
> ```bash
> pkill -f ws_gateway
> ```
> 扫描完成后再重新启动 Gateway。建议在启动 Gateway 前完成所有 CLI 扫描和电机 ID 设置工作。

### 2. 打开 Motorbridge Studio

提示用户访问：https://motorbridge.github.io/motorbridge-studio/

通过网页界面完成零点校准操作。
