# Skyward for the Xiaomi 17 Ultra (nezha)

小米 17 Ultra（`nezha`，型号 25128PNA1C）的 Skyward recovery 成品镜像、配置与适配记录。
上游是 [kirklin/skyward](https://github.com/kirklin/skyward)，这个仓库只针对这一台机器：
**HyperOS `OS4.0.7.0.XPACNXM`（Android 17）**，Thales ST54L eSE 版本。

镜像由 [346540/skyward](https://github.com/346540/skyward) 的 `v0.5.2` 构建（`v0.5.1` + 本机适配），
配置、解密链、四处改动都在本仓库留档。

## 机型

| 项 | 值 |
|---|---|
| 代号 | `nezha` |
| 型号 | 25128PNA1C |
| 系统 | OS4.0.7.0.XPACNXM（Android 17，安全补丁 2026-10-01） |
| 安全元件 | Thales ST54L（`ro.strongbox.model=ST54L-CeSE-1.1`，`ro.special_edition` 为空） |
| 前提 | bootloader 已解锁；只写 `recovery` 分区 |

## 镜像

| 项 | 值 |
|---|---|
| 文件 | `skyward-v0.5.2-nezha-OS4.0.7.0.XPACNXM.img` |
| 大小 | 104,857,600 B |
| sha256 | `9f97d550668ea17a41e1445bb55332e23165df98394392edad9be648c6fb2934` |
| zip | `skyward-v0.5.2-nezha-OS4.0.7.0.XPACNXM.zip`，44,561,643 B |
| zip sha256 | `ab49fc221dc6627cbb2f11d44a46f8b9109343a514df5829d2cee0a90fc3e2e9` |
| stock 基线 | OS4.0.7.0.XPACNXM 整包内 `recovery` 分区，sha256 `43927e2cb10fc0edc6d793e2bec8f4bc4a2e0c51b0a920b65163f4e2118bc302` |

在 [Releases](../../releases) 下载。

## 刷入

```sh
unzip skyward-v0.5.2-nezha-OS4.0.7.0.XPACNXM.zip
fastboot flash recovery_a skyward-v0.5.2-nezha-OS4.0.7.0.XPACNXM.img
fastboot reboot recovery
```

只写 `recovery` 分区，系统分区不动。刷前建议留一份原厂 recovery：

```sh
adb shell su -c "dd if=/dev/block/by-name/recovery$(adb shell getprop ro.boot.slot_suffix) of=/sdcard/stock-recovery.img"
```

进 recovery 后到解密页输 **6 位锁屏 PIN**，`/data` 与 `/sdcard` 就挂上；`暂不解密` 可以直接用其他功能。

## 解密

这台机器的用户密钥同时挂在两个保护者上，**weaver 优先，gatekeeper 兜底**：

| 保护者 | 绑定 | 能不能用 | 说明 |
|---|---|---|---|
| `94a1fd91a7be3906` | weaver 槽 2（`.weaver` = `01 00000002`，`enrolled` 0 字节） | ✅ 可用 | 经 `android.hardware.weaver.IWeaver/default` 读槽，keymint 解 `spblob` |
| `94a1fd91a7be3907` | secdiscardable + gatekeeper（`enrolled` 58 字节） | ❌ 本系统上不可用 | gatekeeper HAL 与 keymint 的 TA 通道返回 `header length: 4294967295`，payload 打不开 |

两个保护者的 salt 都是 `a222b88f831d3acbc0b60480ec589700`，scrypt 参数 `[9,3,1]`，PIN 类型，同一个口令。
细节见 [docs/解密方式.md](docs/解密方式.md)。

## 与上游的四处差别

这些是让这台机器能开机、能解密的全部改动，逐条证据见 [docs/适配记录.md](docs/适配记录.md)：

1. `devices/nezha.toml`：`[stock]` 换到 `OS4.0.7.0.XPACNXM` 的整包地址与分区哈希（构建时校验）。
2. `devices/nezha.toml`：`modules.load` 加 `stm_st54se_gpio`，并把 `/dev/st54spi_gpio`、`/dev/ir_spi` 交给 `system`；
   否则安全元件服务打不开 SPI 通道，`isCardPresent` 回 false，weaver 报 `error 1`。
3. `crates/skyward-programs`：weaver 改以 recovery 自己的用户（root）启动；
   以 `vendor_thales_weaver` 跑时它的 loader 停在 `libvndksupport.so`，`IWeaver/default` 永远注册不上。
4. `crates/skyward-crypto`：`open_user_files` 逐个尝试该用户所有 `spblob`（手机记录的那个优先），
   任一个能开就用它——这台机器记录的恰好是 gatekeeper 那个不可用的。

另外 `crates/skyward` 给 `vendor`/`odm` 上的程序在库路径末尾补 `/mnt/system/system/lib64` 与 `bootstrap`，
并把结果写进内核日志，方便比对。

## 已知限制

- 只在这一台 `nezha`（Thales eSE）上实测；Goodix eSE 版本没有机器验证（上游用 `ro.special_edition` 区分）。
- 改锁屏密码/图案后，Android 会重建保护者，可能又解不开；刷回原厂 recovery 即可。
- 系统 OTA 会覆盖 `recovery` 分区；装完系统记得在 Install 页打开 *Flash Skyward again*。
- 目前基于上游 `v0.5.1`（`43ec6da`），版本号 0.5.2。`v0.3.2` 仍保留在 Releases 里，是最后一次在本机实测通过的版本。

## 自己构建

```sh
git clone https://github.com/346540/skyward && cd skyward
git checkout v0.5.2          # 或按本仓库的 patch 应用改动
cargo build --release -p skyward-pack
target/release/skyward-pack stock   --config devices/nezha.toml
target/release/skyward-pack recovery --stock stock/recovery.img \
  --binary <skyward>  --system-init <system-init>  --config devices/nezha.toml \
  --font resources/fonts/MiSansVF.ttf --mono resources/fonts/KirkMono-Regular.ttf \
  --mono-license resources/fonts/KirkMono-OFL.txt --out out/skyward-nezha.img
```

`skyward` 与 `system-init` 需要用 NDK 交叉编译到 `aarch64-linux-android`；`.github/workflows/build.yml`
在打 tag 时会一次性编好所有机型并发 Release。

## 许可

Skyward 是 GPL-3.0-or-later（见 [LICENSE](LICENSE)）。本仓库的镜像与配置同样遵循该许可，
对应的源码改动见 [skyward-nezha-0.5.2.patch](skyward-nezha-0.5.2.patch) 与 [346540/skyward](https://github.com/346540/skyward)。

上游作者：[kirklin](https://github.com/kirklin/skyward)。感谢。
