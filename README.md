# HP ProDesk 600 G3 SFF Tahoe OpenCore

OpenCore EFI for HP ProDesk 600 G3 SFF running macOS Tahoe.

## Hardware

- Model: HP ProDesk 600 G3 SFF
- Audio codec: Conexant CX20632 (`0x14f15098`)
- Target macOS: Tahoe
- SMBIOS: `iMac20,1`

## Contents

```text
EFI/                              OpenCore EFI
Audio/VoodooHDA-CX20632-speakerfix.kext
                                  Patched VoodooHDA for the internal speaker
```

## Before Use

Generate your own SMBIOS values before booting this EFI. The published `config.plist` has been sanitized.

Replace these fields in `EFI/OC/config.plist`:

- `PlatformInfo -> Generic -> SystemSerialNumber`
- `PlatformInfo -> Generic -> MLB`
- `PlatformInfo -> Generic -> SystemUUID`
- `PlatformInfo -> Generic -> ROM`

## Notes

- No graphics `device-id` injection is used. Injecting a graphics device ID can remove supersampled HiDPI modes; for example, on a 4K display, 2560x1440 HiDPI may disappear and only 1080p HiDPI remains.
- The onboard VGA port on this machine is treated as a digital output path behind the Intel framebuffer rather than a legacy analog VGA device. The EFI keeps the DP-working HD 630 `AAPL,ig-platform-id` (`00001659`), patches only connector 2 to the VGA bridge path, and leaves the native graphics `device-id` untouched.
- SMBIOS is set to `iMac20,1` for Tahoe compatibility. Other SMBIOS models may not be supported by Tahoe.
- Disable Intel Optane optimization for the M.2 slot in BIOS. Otherwise the M.2 NVMe drive may not be detected.
- Set DVMT pre-allocated memory to 64 MB in BIOS.
- USB mapping has already been customized for this machine.
- Tahoe removed AppleHDA, so the usual AppleALC path for analog audio may require restoring or patching system audio components and reducing SIP more aggressively. This EFI instead uses VoodooHDA for analog audio, together with a small CX20632 pin patch for the built-in speaker, so audio can work without taking the full AppleALC + system patch route.

## Graphics

The EFI uses the native HD 630 device ID and does not spoof the graphics device. For the HP ProDesk 600 G3 SFF display outputs, `AAPL,ig-platform-id` is set to `00001659`, matching the layout that keeps the onboard DP path bootable. Connector 2 is patched to `03060A00 00040000 87010000`, which maps the bridged VGA output as a digital DisplayPort-style path.

After changing graphics properties, reset NVRAM once from OpenCore. If either DP or VGA still does not work, test with only one display connected first so the active connector can be identified from IORegistry before adjusting the connector patch.

## Audio

The bundled patched VoodooHDA is for the onboard Conexant CX20632 codec. On this machine, the internal speaker is `nid 31`. Stock VoodooHDA detected it as:

```text
nid 31 0x9217011f Speaker Fixed Analog Internal
```

but merged it into the same output association as the rear line-out/headphone path and then disabled the analog output because of a duplicate pin sequence:

```text
Duplicate pin 15 (31) in association 1! Disabling association.
```

The patched kext changes the internal speaker pin configuration to:

```text
Node:   31
Codec:  0
Config: 0x92170150
```

This places the internal speaker into its own association so VoodooHDA can expose a working analog speaker output.

The patched kext also enables `VoodooHDAEnableVolumeChangeFix` and `VoodooHDAEnableMuteFix`. On this codec, macOS' normal volume slider can move without changing the real analog output level unless VoodooHDA mirrors that change to the PCM mixer.

## Installation Notes

- Copy `EFI` to your EFI partition.
- Use the patched VoodooHDA from `Audio/` for the internal speaker fix.
- Do not use AppleALC and VoodooHDA for the same analog codec at the same time.
- Reset NVRAM after changing OpenCore, SMBIOS, or audio kexts.

## Disclaimer

This configuration is shared as-is for the HP ProDesk 600 G3 SFF. Review every setting before using it on different hardware, especially SMBIOS, USB mapping, storage configuration, and audio routing.

---

# HP ProDesk 600 G3 SFF Tahoe OpenCore 中文说明

这是用于 HP ProDesk 600 G3 SFF 的 OpenCore EFI，目标系统为 macOS Tahoe。

## 硬件信息

- 机型：HP ProDesk 600 G3 SFF
- 声卡：Conexant CX20632 (`0x14f15098`)
- 目标系统：macOS Tahoe
- SMBIOS：`iMac20,1`

## 仓库内容

```text
EFI/                              OpenCore EFI
Audio/VoodooHDA-CX20632-speakerfix.kext
                                  用于内置小扬声器的 VoodooHDA 修补版
```

## 使用前请先修改

发布版本的 `config.plist` 已经脱敏。使用前请自行生成并替换三码。

需要修改 `EFI/OC/config.plist` 中这些字段：

- `PlatformInfo -> Generic -> SystemSerialNumber`
- `PlatformInfo -> Generic -> MLB`
- `PlatformInfo -> Generic -> SystemUUID`
- `PlatformInfo -> Generic -> ROM`

## 注意事项

- 没有对核显进行 `device-id` 注入。注入显卡 device-id 可能会导致超采样 HiDPI 分辨率丢失，例如使用 4K 屏时可能只剩 1080p HiDPI，而没有 2560x1440 HiDPI。
- 这台机器的板载 VGA 接口按 Intel framebuffer 后面的数字输出路径处理，而不是传统模拟 VGA 设备。EFI 保留能让 DP 正常启动的 HD 630 `AAPL,ig-platform-id` (`00001659`)，只把 connector 2 修补到 VGA 桥接路径，不修改原生显卡 `device-id`。
- SMBIOS 使用 `iMac20,1` 以适配 Tahoe。其他 SMBIOS 机型可能不被 Tahoe 支持。
- BIOS 中需要关闭 M.2 接口的 Intel Optane 傲腾优化，否则可能无法识别 M.2 NVMe 硬盘。
- BIOS 中需要将 DVMT 预分配显存调整为 64 MB。
- USB 映射已经针对这台机器定制完成。
- Tahoe 移除了 AppleHDA，因此常规 AppleALC 模拟音频方案可能需要恢复或修补系统音频组件，并更大幅度地降低 SIP。这个 EFI 选择使用 VoodooHDA 处理模拟音频，并针对 CX20632 内置小扬声器加入了一个很小的 pin 修补，从而避免走完整的 AppleALC + 系统补丁路线。

## 核显说明

这个 EFI 保留 HD 630 的原生 device ID，不做显卡设备 ID 仿冒。针对 HP ProDesk 600 G3 SFF 的显示输出，`AAPL,ig-platform-id` 设置为 `00001659`，也就是能让板载 DP 正常启动的布局。connector 2 被修补为 `03060A00 00040000 87010000`，用于把桥接 VGA 输出映射成数字 DisplayPort 风格路径。

修改核显属性后，建议在 OpenCore 中重置一次 NVRAM。如果 DP 或 VGA 仍然不可用，先只连接一个显示器测试，再从 IORegistry 确认当前活动 connector 后继续调整 connector patch。

## 音频说明

仓库内的 VoodooHDA 修补版用于板载 Conexant CX20632 声卡。这台机器的内置小扬声器节点是 `nid 31`。原版 VoodooHDA 能识别出它：

```text
nid 31 0x9217011f Speaker Fixed Analog Internal
```

但会把它和后置 Line-out / 耳机路径合并到同一个输出 association，随后因为 pin sequence 重复而禁用整个模拟输出：

```text
Duplicate pin 15 (31) in association 1! Disabling association.
```

修补版 kext 将内置小扬声器的 pin 配置改为：

```text
Node:   31
Codec:  0
Config: 0x92170150
```

这样可以把内置小扬声器放进独立 association，让 VoodooHDA 正常暴露可用的模拟扬声器输出。

修补版 kext 同时启用了 `VoodooHDAEnableVolumeChangeFix` 和 `VoodooHDAEnableMuteFix`。在这颗声卡上，macOS 系统音量条可能会正常移动，但不会真正改变模拟输出音量；开启该修复后，VoodooHDA 会把系统音量变化同步到 PCM mixer。

## 安装提示

- 将 `EFI` 复制到 EFI 分区。
- 如需使用机箱内置小扬声器，请使用 `Audio/` 目录中的 VoodooHDA 修补版。
- 不要同时为同一个模拟声卡使用 AppleALC 和 VoodooHDA。
- 修改 OpenCore、SMBIOS 或音频 kext 后建议重置 NVRAM。

## 免责声明

此配置按现状分享，仅针对 HP ProDesk 600 G3 SFF。用于其他硬件前请自行检查所有设置，尤其是 SMBIOS、USB 映射、存储配置和音频路由。
