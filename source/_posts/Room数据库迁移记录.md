---
title: Room数据库迁移记录
tags: Android
comments: false
date: 2021-02-19 14:54:37
---
记录一下**Room**数据库迁移过程中遇到的问题。要迁移**Room**数据库，只要需要实现**androidx.room.migration.Migration**即可。
<!--more-->
下面记录三种情况
1. 修改表的结构
```kotlin
object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        //  创建新的临时表
        database.execSQL("CREATE TABLE publish_post_bean_new (uid TEXT NOT NULL DEFAULT '' ,value TEXT NOT NULL DEFAULT '' ,time INTEGER NOT NULL DEFAULT 0, PRIMARY KEY(time))")
        // 复制数据
        database.execSQL("INSERT INTO publish_post_bean_new (uid, value,time) SELECT uid, value ,strftime('%s','now')*1000 FROM publish_post_bean")
        // 删除表结构
        database.execSQL("DROP TABLE publish_post_bean")
        // 临时表名称更改
        database.execSQL("ALTER TABLE publish_post_bean_new RENAME TO publish_post_bean")
    }
}
```
说明：`publish_post_bean`增加了一个`time`的字段，并且修改了主键的类型，先创建了一个临时表，再复制数据，其中复制过程中用`strftime('%s','now')`来获取当前的时间。

2. 只是增加字段
```kotlin
object : Migration(2, 3) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE publish_post_bean ADD COLUMN article TEXT NOT NULL DEFAULT '' ")
        database.execSQL("ALTER TABLE publish_post_bean ADD COLUMN type INTEGER NOT NULL DEFAULT 0 ")
    }
}
```
3. 创建新表
```kotlin
object : Migration(3, 4) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("CREATE TABLE tag_history_bean (tagId TEXT NOT NULL DEFAULT '' ,uid TEXT NOT NULL DEFAULT '' ,value TEXT NOT NULL DEFAULT '' ,time INTEGER NOT NULL DEFAULT 0, PRIMARY KEY(time))")
    }
}
```
以上要注意的是，需要字段的时候，需要添加字段是否是null是否有默认值，比如：` NOT NULL DEFAULT '' `，与你自己定义的bean类型一致。同时可以设置`fallbackToDestructiveMigrationOnDowngrade`来回退数据库升级时发生错误。

