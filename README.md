# 中佳货仓管理系统（中佳 WMS）一键安装器

本仓库是**中佳 WMS** 的官方安装器发布入口（只读）。业务源码不在本仓库，
也不在本仓库的任何 Release 中。

<!--
  公开仓库 Gicce/zhongjia-wms-installer 的 README.md 发布模板（仓库内为
  唯一事实源，发布时替换占位符后整体粘贴；命令行必须与
  InstallerCliContractTest 契约一致——安装器参数即 deploy 参数，官方命令
  不带 deploy 字，兼容「installer.sh deploy --version …」等价写法）。
  占位符：1.0.3 与 1.0.3（V1.0.2 起两者恒同值，
  安装器版本与产品五源版本统一）、f1ddcede322bdea1ee86f251f03e1e15112dcc9e7fbb144b14d56fcf374be837（build-installer.sh
  输出的整文件 SHA256）。
-->

## 一条命令首装（Ubuntu 22.04 / 24.04）

在一台全新公网 Ubuntu 服务器上执行（先落盘校验，绝不 `curl | bash`）：

```bash
curl -fL https://github.com/Gicce/zhongjia-wms-installer/releases/download/v1.0.3/zjwms-installer.sh -o /tmp/zjwms-installer.sh && echo 'f1ddcede322bdea1ee86f251f03e1e15112dcc9e7fbb144b14d56fcf374be837  /tmp/zjwms-installer.sh' | sha256sum -c - && sudo bash /tmp/zjwms-installer.sh --version 1.0.3
```

> 兼容写法：在安装器后先写 deploy 子命令再跟参数（形如
> `zjwms-installer.sh deploy --version 1.0.3`）与上面完全等价——
> 安装器会自动摘除多余的 deploy，不会透传成 `zjwms deploy deploy`。

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
| v1.0.3 | 当前版本（修复 Docker RepoDigests 校验形态缺陷：公网首装在镜像 exact-digest 拉取成功后不再被 FAILED_STEP=MYSQL_IMAGE_VERIFY 误杀；本地校验统一 canonical repository@digest，pull/run 引用不变） |
| v1.0.2 | CLI 契约修复：官方命令不再带 deploy 子命令（同时兼容该写法）。存在 RepoDigests 校验形态缺陷——公网首装镜像拉取成功后仍可能被 MYSQL_IMAGE_VERIFY 误杀——请改用 v1.0.3 |
| v1.0.1 | 首发版本。公开命令多带 deploy 子命令会透传成 `zjwms deploy deploy` 立即失败——请改用 v1.0.3 及以上 |

Release 资产一经发布不再改动；新版本只新增 tag。V1.0.2 起安装器版本与
WMS 产品版本统一。
