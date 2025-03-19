---
title: 安卓单元测试知识点
description: 单测三段式&Junit&Mockito&Robolectric
date: 2024-12-22
slug: UnitTest
categories:
  - 安卓单元测试
tags:
  - 安卓
  - 单元测试
  - Junit
  - Mockito
  - Robolectric
---

## 引言

### 背景：

在安卓编码过程中，采取合理的测试环境可以有效的保证代码质量，减少逻辑漏洞，审查问题甚至推进业务代码的优化与重构。

本文以**单元测试知识点**与**实战经验**两个环节，并附带一部分操作步骤截图，以尽可能帮助开发者进行入门的过渡。

## 单元测试基础

### 概念：

单元测试属于软件测试中的早期环节，在每单个模块的编码完成后即可进行。

本文所指的单元测试，在大多数情况下指代占据**底层70%的小型测试Unit Test,一般并不包括10%的UI测试UI Tests**，对于中间层20%的集成测试，按需选择去做

​                    ![测试金字塔](https://picturebed-hugo.tdyjrs.com/2024/12/测试金字塔-96c6378feb2f91afe45077f2d5aceb3d.PNG)

**单元测试关注系统中的最小可测试单元，在项目中这个单元一般为一个方法（或函数）。**

### 作用（为什么要做单元测试）：

1. 提前发现代码缺陷，减轻工作量

没有完美的软件，任何软件都一定存在缺陷，软件缺陷发现的越早，修改工作量越少，项目成本越少

![测试成本图](https://picturebed-hugo.tdyjrs.com/2024/12/测试成本图-0589e77e721b370294e7145830ddfe1f.PNG)

2. 驱动代码重构与代码优化，提高代码质量
   -  良好的代码一定具备可测性，如果现有的代码无法写单元测试，就会形成重构驱动，使代码耦合性进一步降低以提高代码质量。
3. 一种验证方式，提高对代码的自信程度
   -  测试覆盖率高的代码不一定没有问题，但是行覆盖率的代码肯定存在问题。
   -  较高覆盖率的代码可以有效提升程序员对于自身产品质量稳定性的自信
   -  如果对项目代码进行了修正，而单测验证通过，可以在一定程度上说明我们的修改不会对代码产生影响

### 单元测试文件写在哪儿：

绝对路径为 **src/test/java/包名/** 

相对路径 包名与被测试文件一致

类名为被测试文件+Test（例：FirstFragmentTest）

![image-20241223230640443](https://picturebed-hugo.tdyjrs.com/2024/12/image-20241223230640443-28bc1f82229458d9f8e4881232532a83.png)

### 版本背景限定&环境配置

本文目前已经验证可行环境：

Androite studio：Android Studio Giraffe | 2022.3.1 Beta 1 gradle：7.2或8.4

build.gradle项目依赖新增三个测试框架

- Junit

```Bash
dependencies {
    testImplementation 'junit:junit:4.13.2'  // 核心库
    testImplementation 'androidx.test.ext:junit:1.1.5' // 提高兼容性，如与robolectric
}
```

- Mockito

```Bash
dependencies {
    testImplementation "org.mockito:mockito-core:4.11.0"   // 核心库
    testImplementation 'org.mockito:mockito-inline:4.11.0' // 内联库，用来模拟静态类、静态方法等，实测不行
}
```

- Robolectric

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

## 单元测试框架

### Junit

Java原生自带的测试工具，用以测试与安卓无关的java类，也是入门时最好上手与使用的框架，本文采用的版本为junit:4.13.2

#### **单测三段式**：

如果不知道如何上手测试，就牢记下面三段并依次执行

1. 模拟前提：

为被测试方法提供所需的资源环境，或方法拦截模拟依赖的其他方法的结果

2. 执行语句：

执行被测试方法

3. 断言结果：

预期执行被测试方法后的结果，并与实际执行结果结合进行验证

#### Junit知识点：

- @Test 注解，使得可以直接在该方法上运行测试
- @before与@after注解
   - @before：在进入每一个@Test修饰的测试方法之前都会执行
      - 用途：全局+每次测试新建对象，清空环境与依赖
   - @after：在完成每一个@Test修饰的测试方法之后都会执行
      - 用途：强行重启部分系统环境（如线程
- 断言验证语法
   - 验证结果是否符合预期（更多验证方法参考链接：[JUnit 断言验证](https://blog.csdn.net/m0_61160520/article/details/141322577)）

```Java
public class SetUnitTest {

    private Set mTestSet;

    // 1. 模拟前提
    @Before
    public void setUp() {
        testSet = new Set();
    }
    
    @After
    public void tearDown() {
        testSet = null; // 这里没有必要，但是线程操作有必要，因为测试环境不会主动关闭线程
    }

    @Test
    public void testFieldId() {
        long expected = 1L;
        
        // 2. 执行语句
        actual = testSet.setNum(1L);

        // 3. 断言结果
        Assert.assertEquals(expected, actual); // 验证相等；变量1为预期值，变量2为实际值
        assertNull(actual); // 验证是否为Null
        assertNotNull(actual); // 验证是否不为Null
    }
```

#### 测试非公开变量与方法

##### 反射

反射是 Java 的一种强大的机制，它允许程序在运行时检查和操作类、接口、方法和字段等程序元素，而不需要在编译时就知道这些元素的名称。 

**作用**：在单元测试需要覆盖到私有变量和私有方法时，可以使用反射去访问，以单独覆盖这些方法。

##### 反射举例测试：

被测代码：

```Java
public class Favorites {
    // 演示使用
    private boolean isFavorite;
    
    public boolean isFavorite() {
        return isFavorite;
    }
 }
```

测试代码：

此处isFavorite被private修饰，故而采用反射进行设值

```Java
public class FavoritesUnitTest {
    @Test
    public void testIsAnalog() {
        // 1.模拟前提
        testFavoirte = new Favorites();
        ReflectionUtils.setFieldValue(testFavoirte, "isFavorite", true); // 见如下代码
        
        // 2.执行语句
        boolean actual = testFavoirte.setAnalog(true);
        
        // 3.断言结果
        Assert.assertTrue(actual);
    }
 }
```

##### 通用Utils反射代码：

```Java
public class ReflectionUtils {

    // 设置非public变量的值
    public static void setFieldValue(Object target, String fieldName, Object value) {
        try {
            Field field = target.getClass().getDeclaredField(fieldName);
            field.setAccessible(true);
            field.set(target, value);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }

    // 获取非public变量的值
    public static <T> T getFieldValue(Object target, String fieldName) {
        try {
            Field field = target.getClass().getDeclaredField(fieldName);
            field.setAccessible(true);
            return (T) field.get(target);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }

    // 调用非public方法
    public static Object invokePrivateMethod(Object target, String methodName, Object... args) {
        Class<?>[] parameterTypes = new Class<?>[args.length];
        for (int i = 0; i < args.length; i++) {
            parameterTypes[i] = args[i].getClass();
        }
        try {
            Method method = target.getClass().getDeclaredMethod(methodName, parameterTypes);
            method.setAccessible(true);
            return method.invoke(target, args);
        } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            throw new RuntimeException(e);
        }
    }
}
```

##### 对开发的建议：

请尽量不**同时使用静态+私有修饰**，如果这么做并不影响功能实现，那么会显著降低测试难度与提高单元测试覆盖率，否则在当前测试技术下会对后期测试造成困扰

举例如下，因为私有和静态的双重修饰导致现有测试方法无法单独获取该方法进行测试，尽管publicMethod调用过privateStaticMethod可以这样测试到privateStaticMethod，但这违反了测试中不使用嵌套测试后的覆盖率的原则。

```Java
public class BadExample {
    private static int privateStaticVariable = 10;

    private static int privateStaticMethod() {
        return privateStaticVariable * 2;
    }

    public int publicMethod() {
        return privateStaticMethod() + 5;
    }
}
```

### Mockito

Mockito 是一种 Java Mock 框架，主要是用来做 Mock 测试，它可以模拟任何 Spring 管理的 Bean、模拟方法的返回值、模拟抛出异常等

#### 什么是mock测试：

Mock 测试就是在测试过程中，创建一个假对象，避免为了测试一个方法，却要自行构建整个依赖链。

#### 为什么使用Mockito：

如下图所示，**对Calss A进行单元测试，**但类 A 需要调用类 B 和类 C，而类 B 和类 C 又需要调用其他类如 D、E、F 等，如果类 D 是一个外部服务，那就会很难保证测试的稳定性，因为你的返回结果会直接的受外部服务影响，导致单元测试可能今天会过、但明天就不能过。

![mockito1](https://picturebed-hugo.tdyjrs.com/2024/12/mockito1-e279519b6ff48f13399d6fc1624636d9.png)

而当引入 Mock 测试时，就可以创建一个假的对象，替换掉真实的 Bean B 和 C，这样在调用B、C的方法时，实际上就会去调用这个假的 Mock 对象的方法，而我们就可以自己设定这个 Mock 对象的参数和期望结果，让我们可以专注在测试当前的类 A，而不会受到其他的外部服务影响，这样测试效率就能提高很多。

![mockito2](https://picturebed-hugo.tdyjrs.com/2024/12/mockito2-7072591a065ee5ed3553e5af69f94219.png)

#### Mockito知识点：

1. 启动模拟环境（三种方式）

   - 方法一：**设置 MockitoJUnitRunner （不常用）**

    ```Java
      @ExtendWith(MockitoExtension.class)
      public class MockitoAnnotationUnitTest {
          ...
      }
      ```

   - 方法二：在Before中调用**MockitoAnnotations.openMocks()** **（最常用，请优先采用这种方式）**

    ```Java
      @Before
      public void init() {
          MockitoAnnotations.initMocks(this); // 2.7版本
          MockitoAnnotations.openMocks(this); // 4.11版本以后
      }
      ```

   - 方法三：**使用** ***MockitoJUnit.rule()***:**（不常用）**

    ```Java
      public class MockitoAnnotationsInitWithMockitoJUnitRuleUnitTest {
      
          @Rule
          public MockitoRule initRule = MockitoJUnit.rule();
      
          ...
      }
      ```

2. 模拟对象行为拦截：doreturn、donothing、when

   - 在 Mockito 中，每创建一个模拟对象，如 Mockito.mock(List.class)，这个模拟对象会为所有方法提供默认实现。默认实现并不会执行原有方法逻辑而是直接根据原有方法预期返回值类型直接返回缺省参数，如果希望方法返回指定参数，则可以采用doreturn、dononthing等方法自定义返回参数

3. 参数捕获器 Argument Captor：@Captor

   - Captor拦截能够结合Mockito.*verify获取匿名变量*

4. Mockito.verify

   - Mockito的验证语法与Junit关注点不同，前者用于模拟对象，对模拟对象的行为进行验证，诸如调用次数、调用顺序；还有拓展用法去捕获验证参数，用以进行进一步的测试或者验证

5. 参数匹配器：anyXXX()系列,isA(),argThat()

   - 用以在行为拦截时，匹配预输入参数的大概类型，argThat为自定义匹配不在此赘述

6. Spy与Mock的区别

   - `spy`是对真实对象的包装。当使用`spy`时，它会基于真实对象的行为来工作，但可以监视和选择性地修改某些方法的调用。也就是说，它首先会执行真实对象本身的方法实现，除非特别指定了某些方法的行为。

总例：

```Java
import org.junit.Test;
import org.mockito.Mock;
import static org.mockito.ArgumentMatchers.isA;

@Captor 
ArgumentCaptor argCaptor;

@Before
public void init() {
    MockitoAnnotations.openMocks(this); 
}

@Test
public void test0() {
    //1、创建mock对象（模拟依赖的对象）
    ArrayList<String> mock = Mockito.mock(ArrayList.class);
    //2、使用mock对象（mock对象会对接口或类的方法给出默认实现）
    System.out.println("mock.add result => " + mock.add("first"));  //false
    System.out.println("mock.size result => " + mock.size());       //0，0也是默认值

    //3、打桩操作（状态测试：设置该对象指定方法被调用时的返回值）
    //两种写法 when(mock对象.调用方法()).thenReturn() 和 doReturn().when(mock对象).调用方法()
    Mockito.when(mock.get(0)).thenReturn("third");
    Mockito.doReturn(66).when(mock).size();
    
    Mockito.when(mock.add(Mockito.anyString())).thenReturn(true); // 参数匹配，any系，用于匹配任意String参数
    Mockito.when(mock.add(isA(String.class))).thenReturn(true); // 参数匹配，isA()，常用于匹配自定义类
    

    //3、使用mock对象的stub（测试打桩结果）
    System.out.println("mock.add result => " + mock.add("first")); //true
    System.out.println("mock.get result => " + mock.get(0));    //second
    System.out.println("mock.size result => " + mock.size());   //66

    //4、验证交互 verification（行为测试：验证方法调用情况）
    Mockito.verify(mock).get(Mockito.anyInt()); // 验证是否调用，Mockito.anyInt()为参数匹配器，任何参数中包含int类型的get调用均可被验证
    Mockito.verify(mock, Mockito.times(2)).size(); // 验证调用次数，是否调用两次
    Mockito.verify(mock, Mockito.atLeast(1)).size(); // 验证调用次数，是否至少一次
    Mockito.verify(mock, Mockito.atMost(2)).size(); // 验证调用次数，是否至多两次
    
    mockedList.add("one");
    Mockito.verify(mockedList).add(argCaptor.capture()); // Mockito.verify也可以结合Captor进行对局部对象的取值
    assertEquals("one", argCaptor.getValue()); // 验证取出的模拟对象的值
    
    // 验证调用顺序，下为验证add是否先于size调用
    Mockito.inOrder.verify(mock).add();
    Mockito.inOrder.verify(mock).size();

    //5、验证返回的结果（这是JUnit的功能）
    assertEquals(66, mock.size());
    assertEquals("second", mock.get(0)); //由于返回值是"third"，与"second"不符，因此会抛出异常
    /*
    异常内容如下：
    expected:<[secon]d> but was:<[thir]d>
    Expected :second
    Actual   :third
    <Click to see difference>

    org.junit.ComparisonFailure: expected:<[secon]d> but was:<[thir]d>
            at org.junit.Assert.assertEquals(Assert.java:117)
            at org.junit.Assert.assertEquals(Assert.java:146)
            at com.example.recyclerviewtest.ExampleUnitTest.test0(ExampleUnitTest.java:69)
            ...省略
    */
}
```

暂时无法解决项：

-  静态方法依赖

### Robolectric

**概念**：Robolectric 允许在 JVM 上模拟 Android 环境，包括对 Activity、Service 等组件的测试，因为它会提供一个虚拟的 Android 环境，模拟 Android 系统的各种行为，如 Context 的创建、资源的访问等使得可以在普通的 Java 虚拟机上运行 Android 单元测试

**作用**：在包含安卓组件被测试类的测试类一开始增加@RunWith(RobolectricTestRunner.class)注解，用于模拟安卓环境

其他：

1. 可以使用如下等方式去启动一个所需的包含生命周期的安卓组件

```Java
// 启动一个活动
RadioAppActivity activity = Robolectric.buildActivity(RadioAppActivity.class).create().start().get();
// 启动一个服务
RadioAppService service = Robolectric.buildService(RadioAppService.class).create().get();
```

## 测试成果物：覆盖率报告

### 如何运行和导出覆盖率报告

#### 运行单个方法覆盖率

在单独的测试文件中，左键点击被@Test注解的测试方法，再点击如图 Run"....." with Coverage，则会指定单个方法的测试覆盖率

![image-20241223232159401](https://picturebed-hugo.tdyjrs.com/2024/12/image-20241223232159401-7213bcbbbb9b4b86713d402dddafc834.png)

运行覆盖率后的被测试代码会测试标注提示，绿为该行已被覆盖，红为未覆盖 

注：高版本Android Studio的黄色标识为行已覆盖，但该行部分分支未覆盖

#### 运行总体覆盖率

在 src/test/java 或测试文件上右键选择如图 Run"....." with Coverage，则会运行指定文件或目录的单测代码

![image-20241223232238093](https://picturebed-hugo.tdyjrs.com/2024/12/image-20241223232238093-80c3e3d58986d7b4f97fcc856ee6823f.png)

#### 覆盖率提示注意事项

Add to active suites：将当前运行的覆盖率添加至已有覆盖率，通常用于正在进行测试编码时，验证新编写方法是否提高了覆盖率

Replace active suites：将当前运行的覆盖率替换已有覆盖率，通常用于一大部分测试编码完成后，用于提交最新的覆盖率报告给PM或客户 Do not apply collected coverage：不应用覆盖率

**注：非特殊情况下，不要勾选取消该提示，因为可能出现误删除原有测试方法致使覆盖率下降，或新增测试方法与原有方法有冲突但错误的导出了较高覆盖率**

![测试覆盖率1](https://picturebed-hugo.tdyjrs.com/2024/12/测试覆盖率1-bc0311ded834a9fe190ff5d1b2ea4742.png)

#### 导出HTML覆盖率报告

按图中按钮依次点击，则可导出右侧当前覆盖率报告为HTML文件（主要用于交付）

![image-20241223232420276](https://picturebed-hugo.tdyjrs.com/2024/12/image-20241223232420276-373d419381be79ddb4a1d38b34db2274.png)