# 中佳货仓管理系统（中佳 WMS）一键安装器

本仓库是**中佳 WMS** 的官方安装器发布入口（只读）。业务源码不在本仓库，
也不在本仓库的任何 Release 中。

<!--
  公开仓库 Gicce/zhongjia-wms-installer 的 README.md 发布模板（仓库内为
  唯一事实源，发布时替换占位符后整体粘贴）。V1.0.4 起 Official 命令走
  releases/latest/download 固定 URL——长期 README 不写死任何版本号与
  SHA256，校验依赖随每个 Release 附带的 SHA256SUMS.txt（发布管线保证该
  文件恰好一行「<64位hex>␣␣zjwms-installer.sh」，多行/改写即视为注入，
  官方命令的 wc -l == 1 + grep -qxE 双守卫会拒绝执行）。
  命令行契约与 InstallerCliContractTest 锁定一致：安装器参数即 deploy
  参数；Direct 模式零参数（= Latest 默认）；Proxy 模式唯一参数
  --proxy "$PROXY"。1.0.6 仅用于版本简史表。
-->

## 一条命令首装（Ubuntu 22.04 / 24.04）

在一台全新公网 Ubuntu 服务器上执行（先落盘校验，绝不 `curl | bash`）。
两种模式任选其一，**默认装最新正式版（Latest）**：

### 模式一：直连（GitHub 可直连的服务器）

```bash
curl -fL https://github.com/Gicce/zhongjia-wms-installer/releases/latest/download/zjwms-installer.sh -o /tmp/zjwms-installer.sh \
 && curl -fL https://github.com/Gicce/zhongjia-wms-installer/releases/latest/download/SHA256SUMS.txt -o /tmp/SHA256SUMS.txt \
 && [ "$(wc -l < /tmp/SHA256SUMS.txt)" -eq 1 ] \
 && grep -qxE '[0-9a-f]{64}  zjwms-installer\.sh' /tmp/SHA256SUMS.txt \
 && (cd /tmp && sha256sum -c SHA256SUMS.txt) \
 && sudo bash /tmp/zjwms-installer.sh
```

### 模式二：经 HTTP 代理（GitHub 直连不稳定的服务器）

只改第一行 `PROXY` 为你的代理地址，其余原样复制：

```bash
PROXY='http://代理主机:端口'
curl -x "$PROXY" -fL https://github.com/Gicce/zhongjia-wms-installer/releases/latest/download/zjwms-installer.sh -o /tmp/zjwms-installer.sh \
 && curl -x "$PROXY" -fL https://github.com/Gicce/zhongjia-wms-installer/releases/latest/download/SHA256SUMS.txt -o /tmp/SHA256SUMS.txt \
 && [ "$(wc -l < /tmp/SHA256SUMS.txt)" -eq 1 ] \
 && grep -qxE '[0-9a-f]{64}  zjwms-installer\.sh' /tmp/SHA256SUMS.txt \
 && (cd /tmp && sha256sum -c SHA256SUMS.txt) \
 && sudo bash /tmp/zjwms-installer.sh --proxy "$PROXY"
```

> 代理须为**无认证 http:// 代理**（本版暂不支持 https:// 代理）。部署器会
> 在产生任何系统变更之前，先经该代理完整预检 GitHub / GHCR 连通与镜像
> 数据面可传输性；预检不通过立即停止，服务器零改动。

安装器为单文件自解压脚本，内嵌部署工具链（SHA256 防篡改校验）。网络与
凭据预检全部通过前**零系统副作用**；通过后自动完成：依赖安装 →
MySQL 8.0.39 容器（GHCR 镜像，digest 固定）→ 应用 Release 下载与校验 →
数据库初始化（内置 Fresh Baseline + Flyway 前滚）→ systemd + nginx →
健康检查。

### 需要准备

- 服务器的 sudo 密码（安装器以 root 执行）；
- 一个 **GitHub 只读 Token**（fine-grained PAT，对 `Gicce/zhongjia-wms-release`
  仓库有 Contents: Read 权限）——部署过程中按提示粘贴一次（不回显，不落盘）。

### 首次登录

部署成功后终端会显示一次性管理员口令（同时保存在服务器
`/opt/zhongjia-wms/config/initial-admin.txt`，权限 0600）。首次登录后请立即
修改密码并删除该文件。

## 校验

```bash
sha256sum zjwms-installer.sh        # 与本 Release 的 SHA256SUMS.txt 比对
bash zjwms-installer.sh --self-check
```

## 版本

| 版本 | 说明 |
|---|---|
| Latest（当前 v1.0.6） | Latest 默认安装 + HTTP 代理模式 + 零副作用网络预检（GHCR Blob/CDN 数据面实测通过才动系统）+ 工具链在线升级（--toolchain-only）与 update 三阶段状态提交修复 |
| v1.0.3 | 已过时：GHCR Blob/CDN 数据面直连不稳定的网络下，`docker pull` layer 传输超时导致首装失败（`FAILED_STEP=MYSQL_IMAGE_FETCH`）且无代理出路——请改用最新版 |
| v1.0.2 | Docker RepoDigests 校验修复（digest 拉取成功后被本地判定误杀的问题） |
| v1.0.1 | 首发版本。公开命令多带 deploy 子命令会透传成 `zjwms deploy deploy` 立即失败——请改用最新版 |

Release 资产一经发布不再改动；新版本只新增 tag。安装器版本与 WMS 产品
版本统一。

## 运维手册

安装之外的服务器操作有完整的运维手册，随系统交付提供：

- **部署指南** —— 全新安装、网络模式选择、安装指定旧版本、初始管理员；
- **日常维护指南** —— 在线更新、数据库备份与恢复、部署工具链升级、日常巡检；
- **服务器迁移指南** —— 旧服务器数据完整迁移到新服务器；
- **故障排查指南** —— 按错误标识逐条处置，含"能否安全重试"判断矩阵。

手册全文向系统提供方索取（本仓库不含内部文档）。
