# ZLink

**ZLink** 是一个基于多内核的代理面板节点端程序，二次开发自 [V2bX](https://github.com/wyx2685/V2bX)（V2bX 修改自 XrayR）。

对接 Xboard / 修改版 V2Board（NewV2board 协议），支持 **Vmess / VLESS / Trojan / Shadowsocks / Hysteria2** 多协议，支持 **Xray / sing-box / Hysteria2** 多内核。

## 特性

- 永久开源免费（MPL 2.0）
- 支持 Vmess / VLESS（XTLS / REALITY / Vision）/ Trojan / Shadowsocks / Hysteria1/2
- 支持单实例对接多节点，无需重复启动
- 支持在线设备数上报与**跨节点设备数限制**（依赖面板 UniProxy `alive` / `alivelist` 接口）
- 支持在线 IP 数限制、TCP 连接数限制
- 支持节点端口级别、用户级别限速、动态限速
- 自动申请与续签 TLS 证书（ACME）
- 支持审计规则、自定义 DNS
- 修改配置自动重启实例
- 多内核架构，可通过 build tags 按需编译

## 一键安装

```bash
wget -N https://raw.githubusercontent.com/ZgOm-Pg/ZLink/main/install.sh && bash install.sh
```

支持的系统：Debian / Ubuntu / CentOS / Rocky / AlmaLinux / Alpine / Fedora 等。

## 快速开始

安装完成后编辑配置文件：

```bash
nano /etc/ZLink/config.json
```

### 对接 Xboard / V2Board 节点示例（VLESS + REALITY）

```json
{
  "Log": {
    "Level": "info"
  },
  "Cores": [
    {
      "Type": "xray",
      "Log": {
        "Level": "warning"
      }
    }
  ],
  "Nodes": [
    {
      "Core": "xray",
      "ApiHost": "https://你的面板域名",
      "ApiKey": "节点通信密钥",
      "NodeID": 47,
      "NodeType": "vless",
      "Timeout": 30,
      "ListenIP": "0.0.0.0",
      "SendIP": "0.0.0.0",
      "DeviceOnlineMinTraffic": 200,
      "CertConfig": {
        "CertMode": "none",
        "RejectUnknownSni": false
      }
    }
  ]
}
```

保存后重启服务：

```bash
ZLink restart
```

> 单实例可配置多个 `Nodes`，同时对接多个节点，无需启动多个进程。

## 常用命令

```bash
ZLink start        # 启动
ZLink stop         # 停止
ZLink restart      # 重启
ZLink log          # 查看日志
ZLink update       # 更新到最新版
ZLink uninstall    # 卸载
```

或使用 systemd：

```bash
systemctl start/stop/restart ZLink
journalctl -u ZLink -f
```

## 从 XrayR 迁移

XrayR 已停止维护，ZLink（V2bX 系）是其继任者，且补齐了在线设备数上报（`POST /alive`）与跨节点设备限制（`GET /alivelist`）：

| XrayR 配置 | ZLink 对应 |
|---|---|
| `ApiHost` / `ApiKey` / `NodeID` / `NodeType` | 原样照抄 |
| `VlessFlow: xtls-rprx-vision` | 节点参数由面板下发，无需本地配置 |
| `DeviceLimit: 0`（本地） | 不再需要，设备限制由面板下发自动生效 |
| `GlobalDeviceLimitConfig` | 不再需要，跨节点限制由面板聚合 |

迁移步骤：

1. 安装 ZLink：`wget -N https://raw.githubusercontent.com/ZgOm-Pg/ZLink/main/install.sh && bash install.sh`
2. 按上表填写 `/etc/ZLink/config.json`
3. `ZLink restart` 并确认面板节点在线
4. 确认无误后停用 XrayR：`systemctl disable --now XrayR`

## 从 V2bX 迁移

ZLink 保持 V2bX 配置格式兼容：备份 `/etc/V2bX/config.json`，安装 ZLink 后将配置放入 `/etc/ZLink/config.json` 即可，无需其他修改。

## 构建说明

需要 Go 1.25+：

```bash
GOEXPERIMENT=jsonv2 go build -v -o ZLink -tags "sing xray hysteria2 with_quic with_grpc with_utls with_wireguard with_acme with_gvisor" -trimpath -ldflags "-s -w -buildid="
```

通过 `-tags` 选择要编译的内核：`xray` / `sing` / `hysteria2`。

## 免责声明

- 本项目基于 V2bX（MPL 2.0）二次开发，遵循原许可证
- 仅用于学习研究和合法授权的网络运维场景，使用者需遵守所在地区法律法规
- 使用本项目造成的任何后果由使用者自行承担

## 致谢

- [wyx2685/V2bX](https://github.com/wyx2685/V2bX)
- [XrayR-project/XrayR](https://github.com/XrayR-project/XrayR)
- [XTLS/Xray-core](https://github.com/XTLS/Xray-core)
- [SagerNet/sing-box](https://github.com/SagerNet/sing-box)
- [apernet/hysteria](https://github.com/apernet/hysteria)
