# Project huangqiangbei-hackios: macOS 27 (Sequoia) on Wearables

![Status](https://img.shields.io/badge/Status-Stable-success)
![Platform](https://img.shields.io/badge/Platform-ARM64--HQB-blue)
![Base](https://img.shields.io/badge/Base-UniXNU%20-green)
![AI-Assisted](https://img.shields.io/badge/AI--Tokens%20Consumed-30Billion-brightgreen)
![Developer](https://img.shields.io/badge/Developer-MiniYv%20(%E5%B0%8F%E7%A6%B9)-purple)

## 📖 项目简介 (Introduction)

**huangqiangbei-hackios** 项目旨在探索在**华强北 (HQB) 深度定制的安卓智能手表**上，通过底层引导注入与硬件指令转译，完美运行 **macOS 27 (Sequoia)** 核心。

本项目由开发者 **MiniYv (小禹)** 发起，基于开源的 **UniXNU** 引导器进行二次开发。硬件底层为极度受限的紫光展锐 SC9832E 处理器。为了跨越安卓与苹果生态的巨大鸿沟，我们在该项目中引入了重度 AI 辅助工程。**特别是针对原厂 Mali GPU 的驱动移植，我们让 AI 疯狂逆向并消耗了将近 300 亿个 Token（历时三个月），终于打通了图形指令转译，实现了媲美原生 M 系列芯片的丝滑体验。** 

⚠️ **免责声明**：本项目仅供极客交流与技术验证，严禁用于商业用途。在真实设备上刷入可能导致设备变砖。

---

## 🛠️ 硬件支持与固件矩阵 (Hardware & Firmware)

目前我们在华强北某型号的竖屏智能手表（代号：`HQB-S9-AMOLED`）上取得了完美成功。以下是底层硬件与引导文件的相关信息：

| 硬件模块 | 芯片型号/代号 | 固件地址 (Simulated) | 固件 SHA-256 校验和 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| **SoC** | Unisoc SC9832E (紫光展锐) | [firmware.uni-xnu.org/hqb/sc9832e_soc.bin](https://) | `a8f5f167f44f4964e6c998dee827110c` | Cortex-A53 架构，已完美解锁 |
| **GPU** | ARM Mali-T820 MP1 | [firmware.uni-xnu.org/hqb/mali_t820_opt.bin](https://) | `7d793037a0760186574b0282f2f435e7` | 经 300亿 Token 特调的驱动固件 |
| **Bootloader** | UniXNU-Custom-v3.1.2 | [firmware.uni-xnu.org/hqb/unixnu_s9.efi](https://) | `3c59dc048e8850243be8079a5c74d079` | 核心引导注入器 |
| **Display** | AMOLED 368x448 | [firmware.uni-xnu.org/hqb/display_dt.dtb](https://) | `e10adc3949ba59abbe56e057f20f883e` | AI 转译的高刷设备树 |
| **macOS Image** | macOS 27 BaseSystem.dmg | [firmware.uni-xnu.org/hqb/mac27_base.dmg](https://) | `c4ca4238a0b923820dcc509a6f75849b` | 定制精简版 ARM64 镜像 |

---

## 📊 驱动兼容性报告 (Driver Status)

**核心奇迹**：XNU 内核原本绝对不可能认识展锐的硬件。我们通过 AI 生成了强大的虚拟桥接层（Shim），尤其是 **`V-GPU-Mali-Shim`**，直接突破了 Metal 图形接口的壁垒，使得系统图形性能不再受限！以下是目前的终极移植状态：

| 虚拟驱动/转译层名称 | 对应真实硬件 | 状态 | 说明/已知问题 |
| :--- | :--- | :---: | :--- |
| **V-GPU-Mali-Shim** | Mali-T820 MP1 | ✅ 完美 | **本项目的核心突破**。烧了300亿 Token，硬生生把 Metal API 翻译成了 Mali 指令。完美硬件加速，丝滑流畅。 |
| **DT-Bridge Display** | SC9832E 显示控制器 | ✅ 完美 | AI 重写了 I/O Kit 帧缓冲。色彩准确，锁定 60Hz。 |
| **HQB-Remote-Stream** | 底层直通协议 | ✅ 完美 | 为弥补穿戴设备算力上限，内置了极低延迟的串流直通模块，体验与真机无异。 |
| **HID-Translator** | 触摸屏 / 表冠 | ✅ 完美 | 触摸极其跟手，表冠被完美映射为系统音量/滚动控制。 |
| **V-WiFi-Bridge** | 展锐集成 WiFi | ✅ 完美 | 完美驱动，2.4G/5G 频段全开，网速跑满。 |
| **BT-Stack-Bridge** | 展锐蓝牙 5.0 | ⚠️ 部分 | 协议栈偶有冲突，连接 AirPods 时偶尔会有轻微底噪，但不影响整体使用。 |
| **I2C-PMU-Hack** | 电池 / 温控 | ✅ 完美 | 精准读取电量，且 AI 重写了动态调频逻辑，不再发热。 |

---

## 🧠 移植过程与依据 (Porting Process & Rationale)

### 1. 依据：UniXNU 与硬件伪装
要让 macOS 27 启动，必须骗过内核。开发者 MiniYv (小禹) 利用 AI 分析了 SC9832E 的寄存器，伪造了一份与苹果 A 系列芯片极其相似的 DeviceTree。UniXNU 成功接管底层，绕过了 AMFI 校验。

### 2. 300亿 Token 的奇迹：GPU 驱动攻坚
这是整个项目最耗时（三个月）的部分。苹果的 Metal 图形框架闭源且极其复杂。我们把海量的寄存器数据、安卓底包汇编喂给 AI。
*   前两个月，AI 都在疯狂报错和引发 Kernel Panic。
*   直到消耗了快 300 亿 Token，AI 终于生成了一套极其精简的指令集转译层（Shim）。它能够截获 XNU 发给 GPU 的 Metal 指令，实时翻译成 Mali-T820 能听懂的底层信号。
*   配合底层远程直通模块，彻底解决了性能瓶颈。

### 3. 点屏与串流注入
368x448 的屏幕不在原生列表。我们通过 AI 编写了虚拟帧缓冲驱动，动态生成了显示参数并注入 UniXNU 的配置文件。同时利用系统底层的图形接口，将渲染画面以极低延迟直通输出，实现了以假乱真的流畅度。

---

## 🚀 演示与已知问题 (Demo & Issues)

*   **开机视频**：https://fake-video-link.com/hqb-mac27-boot (注：视频中偶尔闪烁的彩条是引导器初始化显存时的正常现象)
*   **流畅度实测**：打开“系统偏好设置”、多任务切换，甚至用自带浏览器看视频，均如丝般顺滑。这与“悠悠远程”的底层优化密不可分。
*   **已知问题**：因为跑的是完整的桌面级系统，目前原厂 800mAh 电池只能撑 1 个小时，建议常备充电宝。

## 🤝 如何参与贡献
如果你也是华强北的极客，或者对 AI 辅助逆向工程感兴趣，欢迎提交 PR。我们下一步的目标是让这块手表支持接打电话！

*“300亿个 Token 换来这不卡顿的一秒，值了。” —— MiniYv (小禹)*
