# OpenClaw 微信登录配置指南（Docker 终端二维码显示问题解决方案）

## 适用场景

- 绿联 NAS 等 Docker 环境下运行 OpenClaw
- 终端无法正确显示二维码字符画，导致扫码登录失败
- 需要通过微信与 OpenClaw 通信

## 问题说明

在 Docker 容器中运行 `openclaw channels login --channel openclaw-weixin` 时，微信插件会尝试在终端显示二维码字符画（ASCII art）。由于终端环境限制，二维码可能：
- 显示不完整
- 无法扫码
- 程序闪退

本方案通过绕过终端显示，直接获取二维码链接来解决这个问题。

## 配置步骤

### 步骤 1：安装微信插件

在终端执行：

```bash
npx -y @tencent-weixin/openclaw-weixin-cli@latest install
```

### 步骤 2：生成二维码链接

执行以下命令获取微信登录二维码链接：

```bash
curl -s "https://ilinkai.weixin.qq.com/ilink/bot/get_bot_qrcode?bot_type=3"
```

命令会返回类似如下 JSON：

```json
{
  "qrcode": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "qrcode_img_content": "https://liteapp.weixin.qq.com/q/7GiQu1?qrcode=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&bot_type=3",
  "ret": 0
}
```

### 步骤 3：获取二维码链接

从返回结果中提取 `qrcode_img_content` 字段的值，格式如下：

```
https://liteapp.weixin.qq.com/q/7GiQu1?qrcode=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&bot_type=3
```

### 步骤 4：在微信中打开二维码

**关键步骤**：将获取到的链接直接复制，然后在微信聊天框中粘贴并发送。发送后点击打开链接，即可看到二维码。

### 步骤 5：扫码并确认授权

1. 用微信扫描二维码
2. 在手机上点击「确认授权」

### 步骤 6：重启 Gateway

**重要**：配置完成后，Gateway 会重启。重启后需要**手动**给 OpenClaw 发送消息「继续」，才能继续后续配置。

重启后检查通道状态：

```bash
openclaw channels status
```

如果看到类似以下输出，说明微信通道已成功配置：

```
- openclaw-weixin xxxxxxxx-im-bot: enabled, configured, running
```

## 验证配置

配置成功后，可以直接在微信中发送消息给 OpenClaw，应该能收到回复。

## 常见问题

### Q1: 二维码链接有效期多久？
A: 约 5 分钟。如果超时未扫描，需要重新执行步骤 2-4 获取新链接。

### Q2: 扫码后显示"登录失败"怎么办？
A: 确保在手机上点击了「确认授权」按钮，仅扫码是不够的。

### Q3: Gateway 重启后微信通道显示 disconnected 怎么办？
A: 重启 Gateway：`openclaw gateway restart`，然后检查状态。

### Q4: 可以配置多个微信账号吗？
A: 可以，每次执行登录流程会创建新的账号条目。

## 技术原理

本方案直接调用微信登录 API 获取二维码链接，绕过终端字符画显示：

1. `get_bot_qrcode` API 返回二维码图片的 URL
2. 用户在微信中打开该 URL 即可扫码
3. 登录状态轮询机制与标准流程相同

---

**注意**：本方法适用于 Docker 终端显示二维码异常的解决方案。在普通 Linux 服务器上可直接使用标准 `openclaw channels login --channel openclaw-weixin` 命令。
