---
title: 安卓单元测试分享1
description: 什么是单元测试
date: 2024-12-22
slug: UnitTest1
categories:
tags:
---

## 引言

### **背景：**

在安卓编码过程中，采取合理的测试环境可以有效的保证项目组的代码质量，减少逻辑漏洞，审查问题甚至推进业务代码的优化与重构。

本文以**单元测试知识点**与**实战经验**两个环节，并附带一部分操作步骤截图，以尽可能帮助开发者进行入门的过渡。

## 单元测试基础

### **概念：**

单元测试属于软件测试中的早期环节，在每单个模块的编码完成后即可进行。

本文所指的单元测试，在大多数情况下指代占据**底层70%的小型测试Unit Test,一般并不包括10%的****UI****测试UI Tests，**对于中间层20%的集成测试，按需选择去做

 由于项目组覆盖率要求以及覆盖率报告本身的Bug，本文实战经验难点会介绍如何使用单元测试覆盖UI

![测试金字塔](https://picturebed-hugo.tdyjrs.com/2024/12/测试金字塔-96c6378feb2f91afe45077f2d5aceb3d.PNG)

**单元测试关注系统中的最小可测试单元，在项目中这个单元一般为一个方法（或函数）。**

### 作用（为什么要做单元测试）：

1. 提前发现代码缺陷，减轻工作量

没有完美的软件，任何软件都一定存在缺陷，软件缺陷发现的越早，修改工作量越少，项目成本越少

![测试成本图](https://picturebed-hugo.tdyjrs.com/2024/12/测试成本图-0589e77e721b370294e7145830ddfe1f.PNG)

1. 驱动代码重构与代码优化，提高代码质量
   1.  良好的代码一定具备可测性，如果现有的代码无法写单元测试，就会形成重构驱动，使代码耦合性进一步降低以提高代码质量。
2. 一种验证方式，提高项目组对代码的自信程度
   1.  测试覆盖率高的代码不一定没有问题，但是行覆盖率的代码肯定存在问题。
   2.  较高覆盖率的代码可以有效提升项目组对于自身产品质量稳定性的自信
   3.  如果对项目代码进行了修正，而单测验证通过，可以在一定程度上说明我们的修改不会对代码产生影响

### 单元测试文件写在哪儿：

绝对路径为 **src/test/java/包名/** 

相对路径 包名与被测试文件一致

类名为被测试文件+Test（例：FirstFragmentTest）

![image-20241223230640443](https://picturebed-hugo.tdyjrs.com/2024/12/image-20241223230640443-28bc1f82229458d9f8e4881232532a83.png)

### 版本背景限定&环境配置

目前已经验证可行环境：

Androite studio：Android Studio Giraffe | 2022.3.1 Beta 1 gradle：7.2或8.4

build.gradle项目依赖新增三个测试框架

1. Junit

```Bash
dependencies {
    testImplementation 'junit:junit:4.13.2'  // 核心库，应该自带了
    testImplementation 'androidx.test.ext:junit:1.1.5' // 提高兼容性，如与robolectric
}
```

1. Mockito

```Bash
dependencies {
    testImplementation "org.mockito:mockito-core:4.11.0"   // 核心库
    testImplementation 'org.mockito:mockito-inline:4.11.0' // 内联库，用来模拟静态类、静态方法等，实测不行
}
```

1. Robolectric

```Bash
dependencies {
    testImplementation "org.robolectric:robolectric:4.11.1"  // 核心库
    testImplementation "org.robolectric:android-all:10-robolectric-5803371" // 指定需要模拟的安卓api版本环境
}
```

**注意：testImplementation为单元测试依赖，androidTestImplementation为仪器测试依赖，本文只讲单元测试依赖**

### 原则        

1. 一个被测试方法至少有一个专属的测试方法
2. 一个被测试方法有多少个分支，理论上需针对写多少个测试方法
3. 只测试方法本身的逻辑功能，即不要关注被测方法对其他方法的调用，使用拦截等手段模拟这些调用结果