---
title: Android P 静默安装
date: 2019-07-22 14:00:00
tags: [Android]
---
此前低版本静默安装一般分为两种套路：

1. shell 调用 pm 命令
2. 反射调用 PackageManager 的 install 方法

但是在 9.0 上都失效了

分析 PackageInstaller 的源码，和 PackageManager 的源码。发现 PackageManager 多了一个`getPackageInstaller` 的接口，返回了 `PackageInstaller` 对象，再来看一看 PackageInstaller的接口

![](https://s2.ax1x.com/2019/07/19/ZvTSjU.jpg)

初步猜测在 Android P 上采用类似 socket 的方式与 server 端通信完成安装。

# PackageInstaller

查阅官方文档后得知，PackageInstaller 提供了安装、更新以及卸载等功能，其中包括单 APK 和多 APK 安装。

具体的安装行为是通过 PackageInstaller 内部的 Session 完成的。所有的应用都有权限创建这个 Session，但是可能会需要用户的确认才能完成安装（权限不足）。

## Session

创建 Session 可以为其指定参数 `SessionParams`，其中一个作用就是要全部替换还是局部替换 `MODE_FULL_INSTALL` 和 `MODE_INHERIT_EXISTING` 

## 如何安装？

通过 IO 流的方式向 Session 内输送 apk 数据。具体代码可以看下文。需要注意的是，PackageInsatller 对于安装结果回调没有采用普通的函数回调，而是采用 Intent 的方式完成回调，比如 广播。

## 如何在回调中判断是否成功

以广播为例，收到的 Intent 中带有信息，通过`PackageInstaller.EXTRA_STATUS` key 可以获取到安装结果，通常会有 `STATUS_PENDING_USER_ACTION`， `STATUS_SUCCESS`， `STATUS_FAILURE`，`STATUS_FAILURE_ABORTED`， `STATUS_FAILURE_BLOCKED`，`STATUS_FAILURE_CONFLICT`， `STATUS_FAILURE_INCOMPATIBLE`，` STATUS_FAILURE_INVALID`，和 `STATUS_FAILURE_STORAGE`。

# 具体代码

```kotlin
/**
 * 函数入口
 **/
fun installApkInP(apkFilePath: String) {
    val apkFile = File(apkFilePath)
    val packageInstaller = packageManager.packageInstaller
    val sessionParams = PackageInstaller.SessionParams(
        PackageInstaller
        .SessionParams.MODE_FULL_INSTALL
    )
    sessionParams.setSize(apkFile.length())
    val sessionId = createSession(packageInstaller, sessionParams)
    if (sessionId != -1) {
        val copySuccess = copyApkFile(packageInstaller, sessionId, apkFilePath)
        if (copySuccess) {
            install(packageInstaller, sessionId)
        }
    }
}

/**
 * 根据 sessionParams 创建 Session
 **/
private fun createSession(
    packageInstaller: PackageInstaller,
    sessionParams: PackageInstaller.SessionParams
): Int {
    var sessionId = -1
    try {
        sessionId = packageInstaller.createSession(sessionParams)
    } catch (e: IOException) {
        e.printStackTrace()
    }
    return sessionId
}

/**
 * 将 apk 文件输入 session
 **/
private fun copyApkFile(
    packageInstaller: PackageInstaller,
    sessionId: Int, apkFilePath: String
): Boolean {
    var success = false
    val apkFile = File(apkFilePath)
    try {
        packageInstaller.openSession(sessionId).use { session ->
            session.openWrite("app.apk", 0, apkFile.length()).use { out ->
                FileInputStream(apkFile).use { input ->
                    var read: Int
                    val buffer = ByteArray(65536)
                    while (input.read(buffer).also { read = it } != -1) {
                        out.write(buffer, 0, read)
                    }
                    session.fsync(out)
                    success = true
                }
            }
        }
    } catch (e: IOException) {
        e.printStackTrace()
    }
    return success
}

/**
 * 最后提交 session，并且设置回调
 **/
private fun install(packageInstaller: PackageInstaller, sessionId: Int) {
    try {
        packageInstaller.openSession(sessionId).use { session ->
            val intent = Intent(this, InstallResultReceiver::class.java)
            val pendingIntent = PendingIntent.getBroadcast(
                this,
                1, intent,
                PendingIntent.FLAG_UPDATE_CURRENT
            )
            session.commit(pendingIntent.intentSender)
        }
    } catch (e: IOException) {
        e.printStackTrace()
    }
}
```

