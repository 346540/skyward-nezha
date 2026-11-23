# 变更记录

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
