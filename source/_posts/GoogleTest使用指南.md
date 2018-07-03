---
title: GoogleTest使用指南(一)基础
date: 2018-06-12 12:04:17
tags: GoogleTest
categories: 技术
---

## 下载，编译

官方代码托管于[github](https://github.com/google/googletest),使用gitclone拉到本地。

进入googletest/gooletest/msvc目录下，可以直接用vs打开gtest.sln解决方案。

接着直接编译gtest模块，

__若编译失败__,打开“解决方案资源管理器”，右键打开项目“属性”，在C/C++ --> “预处理器”--> “预处理定义”中增加以下行即可：

_VARIADIC_MAX=10;

在debug目录下生成gtestd.lib，将gtestd.lib和 gtest/include/gtest头文件目录包含进入待测试的项目。

在VS工程中，添加c/c++工程中外部头文件及库的基本步骤：

    * 添加工程的头文件目录：工程---属性---配置属性---c/c++---常规---附加包含目录：加上头文件存放目录。

    * 添加文件引用的lib静态库路径：工程---属性---配置属性---链接器---常规---附加库目录：加上lib文件存放目录。然后添加工程引用的lib文件名：工程---属性---配置属性---链接器---输入---附加依赖项：加上lib文件名。

    * 若项目是静态链接库，则在(库管理器->常规)里添加你引用库的库目录和依赖的lib名称。

    * 添加工程引用的dll动态库：把引用的dll放到工程的可执行文件所在的目录下。

## 使用

* 在源文件中包含头文件  

```c++
    #include <gtest/gtest.h>
```

* 编写测试用例

__待测函数__：

```c++
    int Foo(int a,int b) {
        if (b!=0) {
            return a%b;
        }
    else return 0;
}
```

__编写测试用例__：

```c++
TEST(FooTest, Calmod)
{
    EXPECT_EQ(2, Foo(10, 4));
    EXPECT_EQ(3, Foo(10, 7));
}
```

其中，第一个参数为待测函数名首字母大写+Test。
EXPECT_EQ则是GTest断言的一种形式。
断言有两大类。

+ ASSERT_* 当检查点失败时，退出当前函数，被认为是fatal错误。
+ EXPERT_* 当检查点失败时，继续往下执行。  

__运行main函数执行测试__：

```c++
int main(int argc, char* argv[])
{
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

testing::InitGoogleTest(&argc, argv);
gtest的测试案例允许接收一系列的命令行参数，因此，我们将命令行参数传递给gtest，进行一些初始化操作。

**RUN_ALL_TEST()运行源文件内定义的所有测试用例，并根据其返回值判断其所包含的测试用例是否通过。故，一定要接受并返回！！！**

__测试截图__:

![screenshot](https://blog.felixplanet.cn/images/googletest_1.png)