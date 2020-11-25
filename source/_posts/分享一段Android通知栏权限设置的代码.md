---
title: 分享一段Android通知栏权限设置的代码
tags: Android
comments: false
date: 2020-11-25 18:10:15
---

检查是否有通知栏权限
```
NotificationManagerCompat.from(context).areNotificationsEnabled()
```
<!--more-->
打开通知栏权限设置页
```Kotlin
import android.content.Context
import android.content.Intent
import android.net.Uri
import android.os.Build
import android.provider.Settings

object NotificationUtil {
    //调用该方法获取是否开启通知栏权限
    fun goToNotificationSetting(context: Context) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            //这种方案适用于 API 26, 即8.0（含8.0）以上可以用
            try {
                val intent = Intent()
                intent.action = Settings.ACTION_APP_NOTIFICATION_SETTINGS
                intent.putExtra(Settings.EXTRA_APP_PACKAGE, context.packageName)
                intent.putExtra(Settings.EXTRA_CHANNEL_ID, context.applicationInfo.uid)
                context.startActivity(intent)
            } catch (e: Exception) {
                toPermissionSetting(context)
            }
        } else {
            toPermissionSetting(context)
        }
    }

    /**
     * 跳转到权限设置
     *
     * @param activity
     */
    private fun toPermissionSetting(activity: Context) {
        if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.LOLLIPOP_MR1) {
            toSystemConfig(activity)
        } else {
            try {
                toApplicationInfo(activity)
            } catch (e: java.lang.Exception) {
                L.printStackTrace(e)
                toSystemConfig(activity)
            }
        }
    }

    /**
     * 应用信息界面
     *
     * @param activity
     */
    private fun toApplicationInfo(activity: Context) {
        try {
            val localIntent = Intent()
            localIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
            localIntent.action = Settings.ACTION_APPLICATION_DETAILS_SETTINGS
            localIntent.data = Uri.fromParts("package", activity.packageName, null)
            activity.startActivity(localIntent)
        } catch (e: java.lang.Exception) {
            L.printStackTrace(e)
        }
    }

    /**
     * 系统设置界面
     *
     * @param activity
     */
    private fun toSystemConfig(activity: Context) {
        try {
            val intent = Intent(Settings.ACTION_SETTINGS)
            activity.startActivity(intent)
        } catch (e: java.lang.Exception) {
            L.printStackTrace(e)
        }
    }

}
```
