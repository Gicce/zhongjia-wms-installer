# 中佳货仓管理系统（中佳 WMS）一键安装器

本仓库是**中佳 WMS** 的官方安装器发布入口（只读）。业务源码不在本仓库，
也不在本仓库的任何 Release 中。

## 一条命令首装（Ubuntu 22.04 / 24.04）

在一台全新公网 Ubuntu 服务器上执行（先落盘校验，绝不 `curl | bash`）：

```bash
curl -fL https://github.com/Gicce/zhongjia-wms-installer/releases/download/v1.0.1/zjwms-installer.sh -o /tmp/zjwms-installer.sh && echo '0726c4dcd75e5426344365f46a1099751ec500e405eb8d2de00d3578ac30feb4  /tmp/zjwms-installer.sh' | sha256sum -c - && sudo bash /tmp/zjwms-installer.sh deploy --version 1.0.1
```

安装器为单文件自解压脚本，内嵌部署工具链（SHA256 防篡改校验），自动完成：
依赖安装 → MySQL 8.0.39 容器（GHCR 镜像）→ 应用 Release 下载与校验 →
数据库初始化（内置 Fresh Baseline + Flyway 前滚）→ systemd + nginx →
健康检查。

### 需要准备

- 服务器的 sudo 密码（安装器以 root 执行）；
- 一个 **GitHub 只读 Token**（fine-grained PAT，对 `Gicce/zhongjia-wms-release`
  仓库有 Contents: Read 权限）——部署过程中按提示粘贴一次（不回显）。

### 首次登录

部署成功后终端会显示一次性管理员口令（同时保存在服务器
`/opt/zhongjia-wms/config/initial-admin.txt`，权限 0600）。首次登录后请立即
修改密码并删除该文件。

## 校验

```bash
sha256sum zjwms-installer.sh        # 与本仓库 SHA256SUMS.txt 比对
bash zjwms-installer.sh --self-check
```

## 版本

| 版本 | 说明 |
|---|---|
| v1.0.1 | 当前版本（Fresh Baseline 空库自动首装 + 首个管理员一次性口令） |

Release 资产一经发布不再改动；新版本只新增 tag。
