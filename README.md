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
| CPU 调频 | E5PStateMac 0.4.0 控制版；保留 Lilu 和 CpuTscSync |

## 使用前

`EFI/OC/config.plist` 中的 `PlatformInfo → Generic → MLB / ROM / SystemSerialNumber / SystemUUID` 已清空或置零。使用者需要填写自己的机器身份信息。此配置不能直接替代另一台机器的 EFI，USB 映射、ACPI 和硬件驱动应按实际硬件核对。

此提交只同步仓库，未修改运行中电脑的 EFI。Apple 固件更新文件、配置备份、旧版 Kext 备份，以及未被当前配置引用的 CPUFriend 文件均未上传。

## CPU 调频

源码、编译说明和实现限制见 [E5PStateMac](https://github.com/GuaiYiHu/E5PStateMac/tree/v0.4.0)。本分支携带与本机验证时相同的 0.4.0 控制版二进制：

```text
SHA-256: 7a1ed9a7f4143c3f77c979dbbb7306955973cd7a8da3237a0923fd617230b6c3
```

驱动面向这个固定内核配置。启动或唤醒后至少等待 15 秒，并完成全部逻辑核的连续采样自检，才允许控制频率。它保留空闲降频和原厂睿频限制，未开启 `AppleXcpmForceBoost`，未解锁全核睿频或修改电压、功耗上限。未知内核或自检失败时使用原生策略。

本机短时测试中，全核各核心平均约 3.191–3.195 GHz；低负载下观察到约 1.2 GHz 的请求倍频。相同 7-Zip 23.01、20 线程测试，压缩和解压吞吐量相对原生配置分别提高约 8.0% / 9.4%。这不是 Linux 性能完全对齐的证明；长期稳定性、睡眠唤醒和功耗尚未完成验证。

需要停用时，在备份 EFI 后，将 `Kernel → Add` 中 `E5PStateMac.kext` 的 `Enabled` 设为 `false`，然后重启。不要尝试热卸载此驱动。

E5PStateMac 的 BSD-2-Clause 许可见 [docs/E5PStateMac-LICENSE.txt](docs/E5PStateMac-LICENSE.txt)。其他 OpenCore 组件和 Kext 的许可由各自上游项目保留。
