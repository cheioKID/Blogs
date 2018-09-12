---
date: 2018-09-12
title: "Java moveFile IOException Solution"
tags:
    - Java
    - Exception
categories:
    - Java
comment: true
---

在Mac上无误的程序

```java
try {
    FileUtils.moveFile(file, target);//用renameTo方法也可以，但是win里都没有抛出异常，才改用FileUtils.moveFile
    isRenamed = true;
} catch (Exception e) { 
    Log.error(e.toString());
    e.printStackTrace();
    return false;
}
```

到win上抛出异常，不能重命名文件


```java
java.io.IOException: Failed to delete original file 'C:\***\******\************.txt' after copy to 'C:\***\******\************.txt.114110'
        at org.apache.commons.io.FileUtils.moveFile(FileUtils.java:3011)
        at com.example.sms.LocalUtil.renameFile(LocalUtil.java:125)
        at com.example.sms.SendSMS.main(SendSMS.java:152)
```

但是发现如果没有从FTP上下载，重命名是没有问题的

看了一下`FileUtil.moveFile()`的源码

```java
public static void moveFile(final File srcFile, final File destFile) throws IOException {
    if (srcFile == null) {
        throw new NullPointerException("Source must not be null");
    }
    if (destFile == null) {
        throw new NullPointerException("Destination must not be null");
    }
    if (!srcFile.exists()) {
        throw new FileNotFoundException("Source '" + srcFile + "' does not exist");
    }
    if (srcFile.isDirectory()) {
        throw new IOException("Source '" + srcFile + "' is a directory");
    }
    if (destFile.exists()) {
        throw new FileExistsException("Destination '" + destFile + "' already exists");
    }
    if (destFile.isDirectory()) {
        throw new IOException("Destination '" + destFile + "' is a directory");
    }
    final boolean rename = srcFile.renameTo(destFile);
    if (!rename) {
        copyFile(srcFile, destFile);
        if (!srcFile.delete()) {
            FileUtils.deleteQuietly(destFile);
            throw new IOException("Failed to delete original file '" + srcFile + "' after copy to '" + destFile + "'");
        }
    }
}
```

试了一下把`移动`改为`复制`和`沉默删除`两个过程的组合，就可以成功的移动了，真是搞不懂

```java
try {
    FileUtils.copyFile(file, target);
    FileUtils.deleteQuietly(file);
    //FileUtils.moveFile(file, target);
    isRenamed = true;
} catch (Exception e) { 
    Log.error(e.toString());
    e.printStackTrace();
    return false;
}
```

_每次在Mac上写好的程序都要在Windows上面Debug，Windows真难对付_

_花了好多时间搜索中英文结果，最后还是看源码解决了。作为菜鸟感觉源码好精致==，以后有时间多看看优秀的人写的代码。忘记是去年还是今年学过Scala的语法，有机会去学习一下Spark的写法。_