# Sing-box 多协议一键部署脚本

一个强大的 Sing-box 自动化部署工具，支持SS/HY2/TUIC/VLESS Reality/AnyTLS Reality 协议自选部署和线路机 VLESS Reality 中转的完整解决方案。

本仓库是 [caigouzi121380/singbox-deploy](https://github.com/caigouzi121380/singbox-deploy) 的 GitHub fork。新增了凭据文件权限保护、非执行式缓存读取、Reality 握手目标初筛，以及 HY2/TUIC 证书验证和客户端导出。上游目前未声明开源许可证。

---

## ✨ 主要特性

### 🎯 部署机功能

- ✅ **一键安装** - 自动部署 Sing-box 最新服务端
- ✅ **自动生成** - 自动生成 密钥和配置文件，Reality 自选或默认SNI
- ✅ **多系统支持** - 支持 Alpine, Debian, Ubuntu, CentOS, RHEL, Fedora 等操作系统
- ✅ **开机自启** - 自动配置 Systemd / OpenRC 开机自启，崩溃自动拉起服务端
- ✅ **连接 IP** - 自动获取公网 IP 或手动输入 连接IP/DDNS域名 并生成客户端链接
- ✅ **管理工具** - 输入 sb 指令进入管理界面查看节点链接、重置端口、服务端控制查看等功能

### 🔗 线路机功能

- ✅ **一键生成** - 从落地机直接生成线路机安装脚本
- ✅ **Reality 入站** - 自动部署 VLESS Reality 入站
- ✅ **灵活端口** - 支持随机生成端口或手动指定
- ✅ **流量转发** - 自动转发流量到落地机SS节点
- ✅ **完整链接** - 生成可用的 VLESS Reality 客户端链接

## 🙏 特别鸣谢以下商家对本项目的赞助支持

<div align="center">

<table>
  <tr>
    <td align="center" width="220">
      <a href="https://app.kaze.network/" target="_blank">
        <img src="https://app.kaze.network/templates/lagom2/assets/img/logo/logo_big.634794647.svg" width="100" alt="Kaze" />
        <br><sub><b>Kaze</b></sub>
      </a>
    </td>
    <td align="center" width="220">
      <a href="https://yuusei.io/" target="_blank">
        <img src="https://github.com/caigouzi121380/singbox-deploy/blob/main/yuu.svg" width="100" alt="Yuusei Network" />
        <br><sub><b>Yuusei Network</b></sub>
      </a>
    </td>
    <td align="center" width="220">
      <a href="https://lxc.lazycat.wiki" target="_blank">
        <img src="https://lxc.lazycat.wiki/upload/logo2.png" width="100" alt="懒猫云" />
        <br><sub><b>懒猫云</b></sub>
      </a>
  

  </tr>
</table>

</div>

## 安装与配置

### 新装节点

在有 root 权限的 Linux VPS 上使用 Bash 运行。脚本为 Debian、Ubuntu、RHEL 系调用 systemd，为 Alpine 调用 OpenRC。以下命令需要已安装 Git。已在 root shell 中时可省略 `sudo`。

```bash
git clone https://github.com/BassYes/singbox-deploy.git
cd singbox-deploy
sudo bash install-singbox-yyds.sh
```

1. 填写节点名称，再输入要部署的协议编号。编号 `1` 到 `5` 依次对应 Shadowsocks、Hysteria2、TUIC、VLESS Reality、AnyTLS Reality。可用空格选择多项。
2. 选中 HY2 或 TUIC 时，脚本询问是否开启服务端证书验证，首次默认 `Y`。选择 `Y` 时，支持的客户端导出里会带上证书或公钥的 SHA-256 校验值。选择 `N` 时，导出的客户端仍用 TLS 加密，但无法凭证书确认服务端身份。两种协议共用这项导出设置。使用新版 Xray 的 v2rayNG 时选 `Y`。
3. 选中 Shadowsocks 后，选择加密方式。连接地址可填写 VPS 公网 IP 或 DDNS 域名。留空时脚本尝试检测出口 IPv4。如果节点经过端口转发或使用纯 IPv6，填写客户端实际能连接的地址。
4. 选中 VLESS Reality 或 AnyTLS Reality 时，可直接输入 SNI、按回车在 VPS 上初筛候选，或输入 `0` 使用默认域名。初筛检查 TLS 1.3、HTTP/2 和目标站响应。代理是否可用，以客户端网络上的实际 Reality 握手及数据传输为准。
5. 为已选协议填写监听端口。留空会随机生成端口，请确认端口空闲，并在 VPS 防火墙和云安全组放行对应入口。HY2、TUIC 使用 UDP，VLESS Reality、AnyTLS Reality 使用 TCP，Shadowsocks 按实际用途放行 TCP 和 UDP。

服务端配置保存在 `/etc/sing-box/config.json`。安装完成后使用 `sudo sb` 查看链接、配置路径和服务状态。

### 导入客户端

| 客户端 | 导入过程 |
| --- | --- |
| Windows v2rayN 7.25.6 | 在 `sb` 中选择 **查看协议链接**。复制 HY2 或 TUIC 对应的 `v2rayn://outbound/…` 行，再在 v2rayN 中从剪贴板导入。该链接使用 sing-box 核心。 |
| 安卓 v2rayNG 2.3.15 | 开启证书验证时，导入带 `pinSHA256` 的 `hy2://…` 链接。若现有 HY2 证书被标记为 CA，脚本会提示使用 Windows 链接或 SFA 配置。 |
| 安卓 sing-box for Android（SFA） | 将 `/etc/sing-box/sfa-client.json` 私下传到手机。在 SFA 的配置页新建本地配置并导入该 JSON。文件包含已启用的 HY2 和 TUIC，可在客户端切换。 |
| 其他协议 | 从 `sb` 或 `/etc/sing-box/uris.txt` 复制对应的链接。导入行为取决于客户端对协议和 URI 格式的支持。 |

节点链接和客户端 JSON 含有可用于连接的凭据，按私有配置保管。脚本在 `/etc/sing-box` 下以受限权限保存这些文件。客户端校验信息取自当前服务端的公开证书。服务端证书或密钥更换后，应重新导出并导入节点。

### 已有节点更新与重新导出

对使用本项目主脚本装好的节点，下载或拉取本仓库的最新版后，在 VPS 上运行以下命令。该模式收紧文件权限并更新 `sb`，保留现有 sing-box 配置和证书，无需重启服务。

```bash
sudo bash install-singbox-yyds.sh --harden-existing
sudo sb
```

选择 **查看协议链接**，按提示选择是否验证证书。脚本会记住选择，后续也可随时重新导出。普通安装流程会重新生成服务端配置，已有节点使用上面的更新模式。

### 配置中转线路机

在落地 VPS 的 `sb` 菜单中选择 **生成线路机脚本**。脚本使用落地机的 Shadowsocks 入站；尚未启用时菜单会询问是否添加。生成的私有文件包含落地机 SS 凭据。把该文件通过安全通道传到线路机，以 root 运行，选择 VLESS Reality 监听端口，并放行对应的 TCP 端口。客户端导入线路机生成的 VLESS 链接后，流量会由线路机转到落地机。

## 发布验证

- 10 个安装入口和 35 段内嵌 Bash 模板通过语法检查。
- 主脚本在模拟安装中同时启用五种协议。开启和跳过 HY2/TUIC 证书验证时，生成的服务端和 SFA 配置均通过 sing-box 1.14.4 配置检查。解码后的 v2rayN 出站与 SFA 中的对应出站一致。
- 使用 sing-box 1.14.4 在本机通过 HY2 和 TUIC 转发 HTTP。两种验证模式均能连接。开启验证并替换为错误公钥指纹时，两种协议均未能完成请求。
- 在模拟已有节点上，`--harden-existing` 保留原服务端配置及证书验证选择，全程未调用包管理器、网络下载或服务管理器。

本次安装测试模拟了包管理和服务管理，未在真实 VPS 上重装，也未操作桌面或手机客户端界面。
