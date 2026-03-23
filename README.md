# OpenClaw 微信登录 Docker 终端问题解决方案

## 问题描述

在绿联 NAS 等 Docker 环境下运行 OpenClaw 时，执行微信扫码登录命令后，终端无法正确显示二维码字符画（ASCII art），导致无法扫码或程序闪退。

## 解决方案

本方案通过直接调用微信登录 API 获取二维码链接，让用户将链接复制到微信聊天框中发送后打开，从而绕过终端显示问题。

## 快速开始

### 1. 安装微信插件

```bash
npx -y @tencent-weixin/openclaw-weixin-cli@latest install
```

### 2. 获取二维码链接

```bash
curl -s "https://ilinkai.weixin.qq.com/ilink/bot/get_bot_qrcode?bot_type=3"
```

返回示例：
```json
{
  "qrcode": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "qrcode_img_content": "https://liteapp.weixin.qq.com/q/7GiQu1?qrcode=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&bot_type=3",
  "ret": 0
}
```

### 3. 微信中打开二维码

**关键步骤**：
1. 复制返回的 `qrcode_img_content` 链接
2. 在微信聊天框中粘贴并发送
3. 点击打开链接，显示二维码

### 4. 扫码授权

1. 用微信扫描二维码
2. 点击「确认授权」

### 5. 验证

```bash
openclaw channels status
```

看到类似输出即成功：
```
- openclaw-weixin xxxxxxxx-im-bot: enabled, configured, running
```

## 注意事项

- 二维码链接有效期约 5 分钟，超时需重新获取
- Gateway 重启后需手动给 OpenClaw 发送消息继续
- Docker 终端显示异常时使用此方案，普通 Linux 服务器可直接用标准命令

## 文档

详细说明请查看：[微信登录配置指南](./docs/weixin-docker-login.md)

## 适用环境

- 绿联 NAS (DX4600 等)
- 其他 Docker 环境下 OpenClaw
- 终端无法正常显示二维码的场景
