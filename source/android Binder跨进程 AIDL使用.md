---
title: android Binder跨进程 AIDL使用
date: 2016-04-10 15:33:47
tags: android
---
### android中AIDL使用。
<!-- more -->

## 1. Binder通信机制流程(整体框架)
![](http://img.blog.csdn.net/20130529140129511)

## 2. 如何使用AIDL  
### 一. 服务端
- 创建一个专门用来 存放AIDL文件的包 如 `com.kkk.myaidl`
-  在 `com.kkk.myaidl` 包下建一个.aidl文件，如 `IService.aidl`,代码如下。
```java
package com.kkk.myaidl;
import com.kkk.aidl.Person;//很重要，即使在同一个包下也要加import.
interface IServiceaidl{
public int getSum(int num1,int num2);
public Person getPerson();
void setPerson(in Person person);
}
另外，接口中的参数除了aidl支持的类型，其他类型必须标识其方向：到底是输入还是输出抑或两者兼之，用in，out或者inout来表示，上面的代码我们用in标记，因为它是输入型参数。
```
- 创建服务端service
```java
package com.kkk.service;
public class MyService extends Service{
@Override
public IBinder onBind(Intent intent){
    return mbinder;
}
private final IServiceaidl.Stub mbinder = new IServiceaidl.Stub(){
@Override
public int getSum(int num1,int num2){
    return num1+num2;
}
}
}
```

- 在manifests中声明service
```
<service android:name=".MyService" android:enabled="true" android:exported="true">
            <intent-filter>
                <action android:name="com.kkk.service.MyService"/>
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </service>
```
设置action是为了其他应用隐式bindService。通过隐式调用的方式来起activity或者service，需要把category设为default，这是因为，隐式调用的时候，intent中的category默认会被设置为default。

- 传输自定义类型的参数。
    android studio中，新建一个aidl文件，会自动生成一个aidl文件夹。在其中定义需要的类型的.aidl文件。如 `Person.aidl`。
```java
package com.kkk.aidl;
parcelable Person;
```
注意，这里的parcelable是类型，不是Parcelable接口。**接着在jave文件夹下建立一个与aidl中相同的包名,如`com.kkk.aidl`。**在包中新建Person.java，并实现Parcelable接口。一定不能直接把Person.java放在自动生成的aidl文件夹下，否则会出现无法找到Person.java类的错误。

### 二. 客户端
- 将服务端工程中的`com.kkk.myaidl`包整个拷贝到客户端中。双方的aidl包名必须完全一致，否则会报错。
-  新建一个ServiceConnection。
```java
 private ServiceConnection mconnection = new ServiceConnection(){
     @Override  
        public void onServiceDisconnected(ComponentName name)  
        {  
            mIMyService = null;  
        }  

        @Override  
        public void onServiceConnected(ComponentName name, IBinder service)  
        {  
            //通过服务端onBind方法返回的binder对象得到MyService的实例，得到实例就可以调用它的方法了  
             mIServiceaidl =IServiceaidl.Stub.asInterface(service);
        }  
 }
```
- 绑定service。
```java
    Intent intent = new Intent();
    intent.setAction(BIND_SERVICE);
    bindService(intent, mconnection, Context.BIND_AUTO_CREATE);
```
- 调用getSum。
```java
  try {
        int sum = mIServiceaidl.add(1, 2);
        Log.e(TAG, "add: " + sum);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
```

### 参考资料
[任玉刚blog](http://blog.csdn.net/singwhatiwanna/article/details/17041691)
[完整项目](https://github.com/z3jjlzt/AIDLservice)
