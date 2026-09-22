# realme Neo8 (RE6402L1 / RMX8899)

This repository is a root-level device tree for the realme Neo8. It can be
cloned directly into `device/realme/RE6402L1` by the OrangeFox Action Builder.
The platform is Qualcomm SM8850 (`canoe`), and the lunch target is
`twrp_RE6402L1-eng`.

## 设备专属配置

- 镜像刷入列表直接使用 `boot_a` / `boot_b`、`recovery_a` / `recovery_b` 等物理 A/B 分区节点，不依赖无后缀槽位别名。
- `super` 与逻辑分区继续使用 TWRP 的标准动态分区映射流程，`twrp.flags` 不重复创建 `super` 项。
- 支持 `rannki` F2FS 虚拟 SD 卡，存在时挂载到 `/SDKa`。
- 当 recovery 后端没有写入 `tw_active_slot` 时，从 `ro.boot.slot_suffix` 或 `ro.boot.slot` 回填当前槽位。
- WLAN 加载器为 WCN7750/WPSS 冷启动等待最多 45 秒，并在 Android 构建系统将预编译文件权限压成 `0644` 时恢复 `neo8_wifi_hal_client` 的执行权限。
- 保留 Neo8 专属的 QTI KeyMint、TMS/SPU Weaver、OPlus 兼容层、DRM、触摸与 recovery init 配置。

## 镜像分区

`recovery/root/system/etc/twrp.flags` 显示以下物理 A/B 镜像目标：

- `boot_a` / `boot_b`
- `init_boot_a` / `init_boot_b`
- `vendor_boot_a` / `vendor_boot_b`
- `recovery_a` / `recovery_b`
- `dtbo_a` / `dtbo_b`
- `vbmeta_a` / `vbmeta_b`
- `vbmeta_system_a` / `vbmeta_system_b`
- `vbmeta_vendor_a` / `vbmeta_vendor_b`

## OrangeFox Action Builder

Для запуска собственного R12 builder открой GitHub Actions и выбери workflow
`Build OrangeFox R12`, затем нажми `Run workflow`. Он использует текущий commit
репозитория и автоматически размещает дерево в `device/realme/RE6402L1`.

Результат находится в artifact `OrangeFox-R12-RE6402L1`.

Для Android 16 используй workflow `Build OrangeFox 16`. Он синхронизирует
manifest `OrangeFox16/platform_manifest_twrp_aosp` с веткой `twrp-16` и сохраняет
результат в artifact `OrangeFox-16-RE6402L1`. В этой сборке `system_dlkm`
остаётся в конфигурации без R12-совместимого workaround.

Use these workflow inputs in
[OrangeFox-Action-Builder](https://github.com/carlodandan/OrangeFox-Action-Builder):

```text
MANIFEST_BRANCH: 12.1
DEVICE_TREE:     https://github.com/kocetovanton28-dotcom/devicetree_rmx8899
DEVICE_TREE_BRANCH: main
DEVICE_PATH:     device/realme/RE6402L1
DEVICE_NAME:     RE6402L1
BUILD_TARGET:    recovery
```

The tree sets `FOX_AB_DEVICE=1` for the device's A/B partition layout.

Important: the builder currently supports only OrangeFox manifests 11.0 and
12.1, while this tree targets Android 16/API 36 and was written for TWRP 3.7.1.
The workflow inputs above prepare the repository layout and lunch target, but a
successful OrangeFox build still requires an OrangeFox 12.1-compatible port of
the Android 16-specific build and vendor configuration.

## TWRP build

```bash
./scripts/apply-patches.sh /path/to/twrp-source RE6402L1

cd /path/to/twrp-source
source build/envsetup.sh
lunch twrp_RE6402L1-eng
m recoveryimage
```

生成文件位于 `out/target/product/RE6402L1/recovery.img`。
