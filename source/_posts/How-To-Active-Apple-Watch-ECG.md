---
title: ⌚️如何开通 AppleWatch 心电图功能
date: 2019-10-16 16:52:00
tags:
---
众所周知，Apple Watch 从 Series 4 版本开始就支持测量 `心电图` 了，但是在大陆的我，时至今日仍然没能用上这种功能，但是最近在网上冲浪的时候发现有位大佬观察出了如何在大陆开通（[大佬地址](https://hiraku.tw/2019/10/4911/)），蹭大佬的光，我也终于如愿用上了 `心电图` （~~不然心里总觉得花了钱享受不到应有的功能很难受啊~~），先附一张图以证明：

![](1.jpeg)

---

接下来是时候展示真正的技术了：

> **一句话解释：** 开启 ECG 功能后会在手机中写入两个标记，所以通过修改手机的备份，再恢复进手机，即可将 ECG 功能置于开启状态，可以直接使用。

# 必备条件

- 任何支持 Apple Watch Series 4 及以上的 iPhone
- **支持 ECG 地区的 Apple Watch**
- iMazing 软件
- [ECG 激活文件](http://cdn.blog.fxcdev.com/ECG_Active.zip)
- 手机的 `健康` App 必须要有记录

# 测试环境

- 国行 iPhone 11 pro `iOS 13.1.2`
- 港版 Apple Watch Series 4 `WatchOS 6.0.1`
- 大陆 iClound 账号

# 操作步骤

1. 取消手表和手机的配对（为了手表读取我们导入的配置）

2. 用 iMazing 对手机做一次加密备份

   **注意：** 一定要开启加密备份，不然 `健康` 数据不会备份和恢复的，我们的修改也就不起作用了，开启方法见下图：

   ![](2.jpeg)

   ![](3.jpeg)

3. 选择刚刚的备份，选择编辑

   ![](4.jpeg)

4. 在 iMazing 左边往下滑可以看到一个 `可编辑备份`。

5. 按照下图，在 `文件系统` 中进入 `HomeDomain/Library/Preferences` 路径

   ![](5.jpeg)

6. 将之前下载的 `ECG 激活` 解压，得到 `com.apple.private.health.heart-rhythm.plist` 文件

7. 将这个文件复制到 `HomeDomain/Library/Preferences` 路径下，如果提示要覆盖，就直接覆盖。

   ![](6.jpeg)

8. 选择修改过的备份，选择 `恢复至设备`

   ![](7.jpeg)

9. 等待还原完成后和 Apple Watch 重新配对

10. 配对的时候记得选择 `新的 Apple Watch`，不要还原手表备份

配对完成后就可以使用 `心电图` 功能了，尽情享用吧～

