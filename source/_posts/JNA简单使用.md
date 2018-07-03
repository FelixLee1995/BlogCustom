---
title: JNA简单使用
date: 2018-06-12 12:10:52
tags: JNA
categories: 技术
---

## JNA简介

JNA全称Java Native Access，是一个建立在经典的JNI技术之上的Java开源框架（https://github.com/java-native-access/jna）。
JNA提供一组Java工具类用于在运行期动态访问系统本地库（native library：如Window的dll）而不需要编写任何Native/JNI代码。
开发人员只要在一个java接口中描述目标native library的函数与结构，JNA将自动实现Java接口到native function的映射。

[jar下载地址]("https://github.com/java-native-access/jna/releases")

__注意点：Jna只能调用dll中的function，而不能调用method。__ 解决方法，包装一个原生的dll去调用dll中类的method，只要dll能正常使用，即可通过Jna调用包装后的dll中的function。

## 使用

+ 在java中引入jna.jar
+ 创建接口继承com.sun.jna.Library
+ 声明需要调用的接口
+ 使用接口的实例来调用接口中的方法

### 示例

```java
public interface TrApi extends Library {

    TrApi TRADERINSTANCE = Native.loadLibrary("USTPtraderapiAF", TrApi.class);

    void CreateFtdcTraderApi(String pszFlowPath);
}

public static void main(String []args) {
    Trapi api = Trapi.TRADERINSTANCE;
    api.CreateFtdcTraderApi("");
}
```

其中对应的参数需要遵循native<-->java的映射关系。

如下：

| Native Type | Size | Java Type | Common Windows Types |
| - | :-: | :-:  | :-: |
| char | 8-bit integer | byte | BYTE, TCHAR |
| short | 16-bit integer | short | WORD |
| wchar_t | 16/32-bit character | char | TCHAR |
| int | 32-bit integer | int | DWORD |
| int | boolean value | boolean | BOOL |
| long | 32/64-bit integer | NativeLong | LONG |
| long long | 64-bit integer | long | __int64 |
| float | 32-bit FP | float |  |
| double | 64-bit FP | double |  |
| char* | C string | String | LPTCSTR |
| void* | pointer | Pointer | LPVOID, HANDLE, LP|

若传递的是结构体，需要定义结构体d额类结构，继承com.sun.jna.Structure并实现getFieldOrder方法，来返回字段顺序。

__示例：__

```java
public class ApiStruct extends Structure {

    byte[] byteArr = new byte[21];
    int [] intArr = new int[12];

    @Override
    protected List<String> getFieldOrder() {
        List order = new ArrayList<String>();
        order.add("byteArr");
        order.add("intArr");
        return order;
    }
}
```

__注意：在传递参数时，准确的来说是给dll传递了一段内存，并定义了内部的字段顺序，故结构体内的长度需要是确定好的，即结构体内有数组时，一定要先声明大小，分配好内存空间，此时传递的结构体才是完好可用的，否则将会产生 invalid memory  access错误。__

## 回调函数的使用
回调的意义在于dll内部声明一个回调函数的接口，并在需要调用的函数中传入回调函数的指针，就可以在函数中指定的位置调用外部实现接口。

dll中定义：

```c++
typedef void(* MyCallback)(char*errorMsg);

extern "C" EXPORTAPI void orderInsert(OrderField order,Mycallback func);
```

然后在dll中需要使用的地方调用 Mycallback

java中需要在jna接口中定义回调接口如下：

```java
void orderInsert(OrderField,Callbackfunc);

public interface Callbackfunc extends Callback {
    public void callback_fun(String msg);
}

public class CallbackfuncImpl implements Callbackfunc {

    @Override
    public void callback_fun(String msg) {
        System.out.println("Message:" + msg);
    }
}

public static void main(String []args) {
    API api = api.instance;
    API.CallbackfuncImpl func = new API.Callbackfuncimpl();
    api.orderInsert(new OrderField(),func);
}
```
