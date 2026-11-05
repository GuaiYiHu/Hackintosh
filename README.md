# X99 / Xeon E5-2666 v3 / RX 560

本分支 `x99-v.1.41-RX560` 保存这台 X99 电脑的 OpenCore EFI。2026-11-05 按实际启动分区同步，保留原分支历史。

| 项目 | 当前配置 |
| --- | --- |
| CPU | Xeon E5-2666 v3，10 核 20 线程，无全核睿频解锁 BIOS |
| 主板 | X99；具体型号未确认 |
| 显卡 | Radeon RX 560 |
| 内存 | 64 GB |
| SMBIOS | MacPro7,1 |
| 已验证系统 | macOS 26.7.2，build 25G332；Darwin 25.6.0 |
| OpenCore | 1.0.8 RELEASE，与配套 UEFI 驱动一起更新 |
| CPU 调频 | E5PStateMac 0.4.0 控制版；保留 Lilu 和 CpuTscSync |

## 2026-11-05 引导与驱动升级

已写入 EFI 并通过文件哈希与 `ocvalidate 1.0.8` 校验。2026-11-06 完成升级后的重启核对：7 个更新的 Kext 均按新版本加载，Lilu、CpuTscSync 与 E5PStateMac 的预期版本也已加载。

66 秒调频复测通过：20 个逻辑核连续采样，满载各核平均 3.168–3.176 GHz，整体平均约 3.171 GHz；低负载忙碌频率采样最低约 1.30 GHz，请求倍频最低为 12（按 100 MHz BCLK 为 1.2 GHz）。E5PStateMac 保持 `controlling`，20 核采样自检掩码完整，未观察到驱动故障或写入回退。频率来自驱动 APERF/MPERF 估算，仅使用 250 ms 内的新鲜样本。

内建输入/输出、显示器音频与 USB 设备可枚举；未发现本次启动后的内核崩溃报告。此次未逐个测试 USB 端口、实际音频播放或睡眠唤醒，也未重新进行吞吐量对比。

| 组件 | 升级 |
| --- | --- |
| OpenCore 与配套 UEFI 驱动 | 1.0.7 → 1.0.8 |
| VirtualSMC / SMCProcessor / SMCSuperIO | 1.3.7 → 1.3.8 |
| BlueToolFixup | 2.7.1 → 2.7.2 |
| AppleALC | 1.9.6 → 1.9.8 |
| WhateverGreen | 1.7.0 → 1.7.1 |
| SimpleHDA | 1.0.0 → 1.0.1 |

Lilu、CpuTscSync、RestrictEvents、USBToolBox、RealtekRTL8111 已与最新稳定版二进制核对一致；AMFIPass 和两项 RtWlan 驱动也已核对上游版本。保留本机 UTBMap、XHCI 定制项、ACPI 与 E5PStateMac。

原 OpenCore 1.0.7 不支持配置中的六项 `UEFI → Drivers → HideVerbose` 字段，会出现 `No schema for HideVerbose`。此次配套升级为支持该字段的 [OpenCore 1.0.8](https://github.com/acidanthera/OpenCorePkg/releases/tag/1.0.8)，配置使用相同版本的校验器验证。SimpleHDA 1.0.1 要求的 `DisableLinkeditJettison=true` 已存在，保留原启动参数。

## 使用前

`EFI/OC/config.plist` 中的 `PlatformInfo → Generic → MLB / ROM / SystemSerialNumber / SystemUUID` 已清空或置零。使用者需要填写自己的机器身份信息。此配置不能直接替代另一台机器的 EFI，USB 映射、ACPI 和硬件驱动应按实际硬件核对。

此仓库保存脱敏后的 EFI；Git 操作不改动运行中电脑的启动分区。Apple 固件更新文件、配置备份、旧版 Kext 备份，以及未被当前配置引用的 CPUFriend 文件均未上传。

## CPU 调频

源码、编译说明和实现限制见 [E5PStateMac](https://github.com/GuaiYiHu/E5PStateMac/tree/v0.4.0)。本分支携带与本机验证时相同的 0.4.0 控制版二进制：

```text
SHA-256: 7a1ed9a7f4143c3f77c979dbbb7306955973cd7a8da3237a0923fd617230b6c3
```

驱动面向这个固定内核配置。启动或唤醒后至少等待 15 秒，并完成全部逻辑核的连续采样自检，才允许控制频率。它保留空闲降频和原厂睿频限制，未开启 `AppleXcpmForceBoost`，未解锁全核睿频或修改电压、功耗上限。未知内核或自检失败时使用原生策略。

2026-11-05 首次 0.4.0 短时测试中，全核各核心平均约 3.191–3.195 GHz；低负载下观察到约 1.2 GHz 的请求倍频。相同 7-Zip 23.01、20 线程测试，压缩和解压吞吐量相对原生配置分别提高约 8.0% / 9.4%。这不是 Linux 性能完全对齐的证明；长期稳定性、睡眠唤醒和功耗尚未完成验证。

需要停用时，在备份 EFI 后，将 `Kernel → Add` 中 `E5PStateMac.kext` 的 `Enabled` 设为 `false`，然后重启。不要尝试热卸载此驱动。

E5PStateMac 的 BSD-2-Clause 许可见 [docs/E5PStateMac-LICENSE.txt](docs/E5PStateMac-LICENSE.txt)。其他 OpenCore 组件和 Kext 的许可由各自上游项目保留。
