# X99 / Xeon E5-2666 v3 / RX 560

本分支 `x99-v.1.41-RX560` 保存这台 X99 电脑的 OpenCore EFI。2026-11-07 按实际启动分区只读采集并脱敏同步，保留原分支历史。

| 项目 | 当前配置 |
| --- | --- |
| CPU | Xeon E5-2666 v3，10 核 20 线程，无全核睿频解锁 BIOS |
| 主板 | X99；具体型号未确认 |
| 显卡 | Radeon RX 560 |
| 内存 | 64 GB |
| SMBIOS | MacPro7,1 |
| 本轮已验证系统 | macOS 15.8.2，build 24H232；Darwin 24.6.0 |
| 历史已验证系统 | macOS 26.7.2，build 25G332；Darwin 25.6.0 |
| OpenCore | 1.0.8 RELEASE，与配套 UEFI 驱动一起更新 |
| CPU 调频 | Sequoia 使用 E5PStateMac 0.5.1 控制版；保留原 26 版 0.4.0、Lilu 和 CpuTscSync |

## 2026-11-07 Sequoia 调频与启动策略

加入 `E5PStateMacSequoia.kext` 0.5.1，限定 `MinKernel = MaxKernel = 24.6.0`；原 `E5PStateMac.kext` 0.4.0 仍限定 Darwin 25 及以上注入，两者范围互斥。OpenCore 继续为 1.0.8。除清空四项机器身份字段外，发布的配置与当前 EFI 一致；发布的其他引导文件与当前 EFI 字节一致，`ocvalidate 1.0.8` 校验通过。

本轮启动加载了预期的 0.5.1 模块，默认额外等待为 0 秒。驱动服务在内核 uptime 约 0.94 秒启动，首次通过全部控制门控是在 uptime 约 12.71 秒，即服务启动约 11.77 秒后；全核资格掩码为 `0xFFFFF`。该记录表示首次允许接管的时刻，没有单独记录第一条实际硬件覆盖的时间戳。耗时取决于各核采样与健康条件，不是固定 15 秒或固定 11.77 秒。

20 线程整数满载下，各逻辑核新鲜高负载样本平均约 3.177–3.184 GHz，整体约 3.180 GHz；powermetrics 稳定满载窗口核对约 3.174 GHz。恢复轻载后请求最低 12 倍频，本轮实际有效频率估计最低约 1.278 GHz。新增 40,380 个有效样本，15,549 次控制覆盖全部成功读回，故障和回退均为 0。此次没有进行睡眠唤醒、长期稳定性或吞吐 A/B 测试。

当前 boot-args 未加入 `-e5pmwait15`。需要保留额外预热时可将该 flag 加入实际启动参数；它在自检之外增加至少 15 秒等待，不会跳过自检。该 flag 的 15 秒路径已通过主机回归及静态检查，尚未单独重启验收。详细方法见 [0.5.1 启动与调频验证](https://github.com/GuaiYiHu/E5PStateMac/blob/main/docs/startup-policy.md)。

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

源码、编译说明和实现限制见 [E5PStateMac](https://github.com/GuaiYiHu/E5PStateMac)。本分支同时保留两个按系统范围互斥注入的控制版；驱动内部还校验固定内核 UUID、函数指纹及 CPU 身份，注入范围不代表支持范围内的所有内核均可控制。

| 文件 | 目标与策略 | 主二进制 SHA-256 |
| --- | --- | --- |
| `E5PStateMacSequoia.kext` 0.5.1 | macOS 15.8.2 / 24H232；默认自检通过即允许接管，`-e5pmwait15` 可选额外预热 | `42f45cff796f5482ef11f96ddbd2fbf90e181cf9ff3984bacc6311329b44010f` |
| `E5PStateMac.kext` 0.4.0 | macOS 26.7.2 / 25G332；至少 15 秒预热及全核自检 | `7a1ed9a7f4143c3f77c979dbbb7306955973cd7a8da3237a0923fd617230b6c3` |

两版均保留全部 20 个逻辑核连续采样自检、低负载降频和原厂睿频限制；未知内核或自检失败时使用原生策略。未开启 `AppleXcpmForceBoost`，未解锁全核睿频或修改电压、功耗上限。

2026-11-05 首次 0.4.0 短时测试中，全核各核心平均约 3.191–3.195 GHz；低负载下观察到约 1.2 GHz 的请求倍频。相同 7-Zip 23.01、20 线程测试，压缩和解压吞吐量相对原生配置分别提高约 8.0% / 9.4%。这不是 Linux 性能完全对齐的证明；长期稳定性、睡眠唤醒和功耗尚未完成验证。

需要停用时可使用启动参数 `-e5pmoff`，或将 `Kernel → Add` 中当前系统对应的 E5PStateMac 条目 `Enabled` 设为 `false`，然后重启。不要尝试热卸载此驱动。

E5PStateMac 的 BSD-2-Clause 许可见 [docs/E5PStateMac-LICENSE.txt](docs/E5PStateMac-LICENSE.txt)。其他 OpenCore 组件和 Kext 的许可由各自上游项目保留。
