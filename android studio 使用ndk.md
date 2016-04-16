---
title: android studio 使用ndk
date: 2016-04-13 16:40:26
tags: java
---
### android studio使用ndk简介。
<!-- more -->

### 准备工作
 - 下载ndk。[下载地址](http://androiddevtools.cn/)

### 新建一个NdkDemo。
- 配置ndk，file->Project Structure。

![](http://ww1.sinaimg.cn/mw690/5a65f7d1gw1f2v5od9xbaj211v0lcdmv.jpg)

- 在gradle.properties文件追加`android.useDeprecatedNdk=true`

![](http://ww4.sinaimg.cn/mw690/5a65f7d1gw1f2v5odsipvj211v0lcdn1.jpg)
- 新建`com.kkk.jni`包用于放置含有native方法的类。在包中新建jniMathKit.class。

```java
package com.kkk.jni;

public class jniMathKit {
    public native int getSum(int a,int b);//get a+b
    static {
        System.loadLibrary("jniDemo");//与build.gradle中务必一致。这里注意,不要以lib开头,否则会出现类初始化错误，我也不知道原因。
    }
}

```
- 新建jni文件夹。右击java文件夹New->Folder->JNI Folder。

![](http://ww2.sinaimg.cn/mw690/5a65f7d1gw1f2v5oeud4oj20l80fvq7j.jpg)

- 点击Rebuild Project,切换目录结构Andorid——>Project。如果在../app/build/下有intermediates继续下一步。

- 打开 终端 terminal，进入到../app/build/intermediates/classes/debug/ 执行`javah com.kkk.jni.jniMathKit`,可以发现在/debug文件夹下多个一个.h文件。把.h文件复制到jni文件夹（与java文件夹同层）中。

![](http://ww2.sinaimg.cn/mw690/5a65f7d1gw1f2v5oealv0j20900c3jsg.jpg)


- 在对应的module的build.gradle文件中defaultConfig中添加ndk配置

```java
  defaultConfig {
        applicationId "com.kkk.ndkdemo"
        minSdkVersion 15
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
        ndk {
            moduleName "jniDemo"
        }
    }
```
- **再次Rebuild Project**。

- 打开jniMathKit,把鼠标移动到getSum上，alt+enter，可以发现多一个Create function Java_com_kkk_jni_jniMathKit_getSum选项，点击创建。此时会创建一个jniDemo.c类，实现.h中的方法如下。

```java
#include "com_kkk_jni_jniMathKit.h"

JNIEXPORT jint JNICALL Java_com_kkk_jni_jniMathKit_getSum
        (JNIEnv *env, jobject jobject, jint a, jint b){
    return a+b;
}
```
- 最后在mainAcitity中调用。编译运行。
```java
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        jniMathKit jniMathKit= new jniMathKit();
        Log.e("sb", "sum :"+ jniMathKit.getSum(5,6) );
    }

```
### 参考资料
[完整项目](https://github.com/z3jjlzt/NdkDemo)
