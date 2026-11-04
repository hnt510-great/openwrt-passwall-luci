# Passwall LuCI feed

本仓库是 [hnt510-great/openwrt](https://github.com/hnt510-great/openwrt) 定制主树使用的私有源码 feed。
项目面向 x86_64/glibc、APK、原生 systemd 和 GNOME 桌面；软件选择以主树
保存的完整配置为准。

| 项目 | 值 |
| --- | --- |
| Feed 名称 | `passwall_luci` |
| 本仓库分支 | `main` |
| 原始上游 | [https://github.com/Openwrt-Passwall/openwrt-passwall](https://github.com/Openwrt-Passwall/openwrt-passwall)（`main`） |

本 feed 提供 `luci-app-passwall` 的管理页面、服务脚本、配置及翻译，配合
`passwall_packages` 中的代理核心和 DNS 工具使用。当前仓库保存本项目的
集成历史，应与私有主树中的依赖和版本规则一起维护。

## 与主树的集成

- 当前镜像保留 Passwall 为默认代理方案；`daed` 默认停用，两者的透明代理
  不应同时启用。切换流程见主树的 daed 文档。
- LuCI、UCI、`/etc/init.d/passwall` 与 procd 服务接口保留。原生 systemd 下，
  进程监督由主树的 procd/systemd 集成处理。
- 主树沿用 LuCI Git 修订版本机制，分别计算 Passwall 代码和翻译版本，
  即使上游没有手动提升版本号，已提交的对应源码变化也能获得新版本。
- 未提交的运行时文件修改不能依赖 Git 修订号获得新的 APK 身份；发布前仍需
  保证包版本递增。

使用主树已经配置好的两个 Passwall feeds 即可，无需删除 `packages` 中的
配方，也无需额外克隆一份到 `package/`。

## 获取与构建

先在构建机登录有权访问全部私有仓库的 GitHub 账号，再获取完整主树：

```sh
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
git clone https://github.com/hnt510-great/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

主树 `feeds.conf.default` 已包含：

```text
src-git passwall_luci https://github.com/hnt510-great/openwrt-passwall-luci.git;main
```

在现有主树中只更新本 feed，可执行：

```sh
./scripts/feeds update passwall_luci
./scripts/feeds install -a -p passwall_luci
```

先在主树配置中选择所需包并运行 `make defconfig`。单包构建示例：

```sh
make package/feeds/passwall_luci/luci-app-passwall/compile V=s
```

以上命令均从主树根目录运行。保留已有 `.config`；新 checkout 的配置恢复
步骤见 [本地构建指南](https://github.com/hnt510-great/openwrt/blob/main/docs/local-build.md)。`install` 安装的是构建配方链接，
不会在设备上安装软件。完整 `update -a` 还会检查并可能修改主树的 custom 配方。

## 维护与验证

`origin` 指向本私有仓库。新 clone 需要合并原始上游时，先添加一次 `upstream`：

```sh
git -C feeds/passwall_luci remote add upstream https://github.com/Openwrt-Passwall/openwrt-passwall.git
git -C feeds/passwall_luci fetch upstream
git -C feeds/passwall_luci merge upstream/main
# 审阅、解决冲突并完成相关验证后推送
git -C feeds/passwall_luci push origin main
```

已有 `upstream` 时跳过添加命令。`scripts/feeds update` 拉取本私有分支，
不会自动合并原始上游。本地集成修改与主树的提交分别管理，保留现有提交、
stash 和未提交文件。主树的 `patches/feeds/` 和功能清单是历史备份，
不要向已经包含修改的 feed 重复应用补丁。

涉及已发布 APK 内容的修改必须产生新包版本，不能覆盖同一包身份。源码
检查、包构建、VM 与物理设备验证应分别记录；本文汇总已有设计和验证记录，
不新增构建或设备验证结论。私有源码托管说明见
[GitHub 源码托管](https://github.com/hnt510-great/openwrt/blob/main/docs/github-source-hosting.md)。

## 设计与验证文档

- [daed 与 Passwall 切换](https://github.com/hnt510-great/openwrt/blob/main/docs/daed.md)
- [原生服务监督](https://github.com/hnt510-great/openwrt/blob/main/docs/systemd-native-supervision.md)
- [Passwall cgroup 验证](https://github.com/hnt510-great/openwrt/blob/main/docs/systemd-cgroups-validation.md)
- [APK 发布与源码版本](https://github.com/hnt510-great/openwrt/blob/main/docs/lan-apk-feeds.md)

## 许可与上游

保留上游版权和许可。各软件包源码的许可按其配方 `PKG_LICENSE` 及源码声明执行。
仓库许可声明见 [LICENSE](LICENSE)。
