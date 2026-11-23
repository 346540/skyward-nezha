# 变更记录

## v0.5.2 — 跟上上游 0.5.1

基于上游 `v0.5.1`（commit `43ec6da`），把 v0.3.2 的四条改动重新落到新代码上：

- `devices/nezha.toml`：`[stock]` 仍指 `OS4.0.7.0.XPACNXM` 整包与 `43927e2c…bc302`；
  `modules.load` 加 `stm_st54se_gpio`；`[[owners]]` 交出 `/dev/st54spi_gpio`、`/dev/ir_spi`。
- `crates/skyward-programs`：weaver 以 recovery 自己的用户启动（Thales 与 Goodix 两种都一样）。
- `crates/skyward-crypto`：`open_user_files` 遍历全部 `spblob`，记录的那个优先。
- `crates/skyward-bootimg`：有 `lz4` 命令就用它，没有就地按 legacy 帧压。
- `crates/skyward`：`vendor`/`odm` 上的程序库路径末尾补安装系统的两个目录，并写入 kmsg。
- 版本 0.5.1 → 0.5.2。

镜像：

```
zip    skyward-v0.5.2-nezha-OS4.0.7.0.XPACNXM.zip  sha256 ab49fc221dc6627cbb2f11d44a46f8b9109343a514df5829d2cee0a90fc3e2e9
img    skyward-v0.5.2-nezha-OS4.0.7.0.XPACNXM.img  sha256 9f97d550668ea17a41e1445bb55332e23165df98394392edad9be648c6fb2934
```

**已上机实测（2026-11-23）**：`nezha` + OS4.0.7.0.XPACNXM，解密页输 6 位 PIN 正常打开 `/data`、`/sdcard`，触摸、WLAN、MTP、振动、fastbootd 全部正常。0.5.x 改过的 binder 层与 weaver 分支在这台机器上没有引入回归。

## v0.3.2 — 首个 17 Ultra 专用版本

基于上游 `v0.3.1`（commit `c5e09f6`），针对 `nezha` / `OS4.0.7.0.XPACNXM` 的改动：

- `devices/nezha.toml`：`[stock]` 指向 `OS4.0.7.0.XPACNXM` 整包与包内 `recovery` 分区哈希
  `43927e2c…bc302`，构建时校验。
- `devices/nezha.toml`：`modules.load` 增加 `stm_st54se_gpio`；`[[owners]]` 把
  `/dev/st54spi_gpio`、`/dev/ir_spi` 交给 `system:system 0660`。
  → 安全元件服务能打开 SPI 通道，`isCardPresent` 不再回 false。
- `crates/skyward-programs`：weaver 以 recovery 自己的用户启动。
  → 不再停在 `libvndksupport.so`，`android.hardware.weaver.IWeaver/default` 正常注册。
- `crates/skyward-crypto`：`open_user_files` 遍历该用户全部 `spblob`（记录的那个优先）。
  → 记录的保护者不可用时自动落到另一个。
- `crates/skyward`：`vendor`/`odm` 上的程序在库路径末尾补安装系统的
  `/mnt/system/system/lib64` 与 `bootstrap`，并写入 kmsg。
- 版本号 0.3.1 → 0.3.2（tag 必须与 workspace version 一致，CI 才会发 Release）。

实测：`nezha`（Thales ST54L eSE）+ `OS4.0.7.0.XPACNXM`，解锁页输 6 位 PIN 后 `/data`、`/sdcard` 可读，
触摸、WLAN、MTP、振动、fastbootd 正常。

镜像：

```
zip    skyward-v0.3.2-nezha-OS4.0.7.0.XPACNXM.zip  sha256 c8fc37821c26ca8030bfc0dda35d29d06b2a3ee6eb5972d27f9a4e46712e3137
img    skyward-v0.3.2-nezha-OS4.0.7.0.XPACNXM.img  sha256 9d6cb2d507850893d2e9396ba80dc64ca2ac6f6d104a4c758e307f87e71b1cf7
```
