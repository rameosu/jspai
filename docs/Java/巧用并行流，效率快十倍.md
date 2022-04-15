# 巧用并行流，效率快十倍

Java 的 Stream 流操作相信大家都不陌生，用 Stream 来操作集合那是相当的顺滑简单，让你爱不释手。但是根据我这么多年的开发经历和代码审查的经历，大部分人都只掌握了`stream()`方法的操作，而不知道并行流 `parallelStream()`。

今天我们就来剖析一下并行流的使用技巧。

## 简介

Java 8 引入了 Stream API，它可以轻松地将集合作为数据流进行迭代。

Java 中的流只是数据源的包装器，允许我们以方便的方式对数据执行批量操作。

它不存储数据或对基础数据源进行任何更改。相反，它增加了对数据管道上函数式操作的支持。

使用 `parallelStream()` 可以创建并行执行的迭代器，使得利用多个处理器内核的流变得异常的简单。

有多简单？看下面的一行代码就知道了：

```java
userList.parallelStream().forEach(user -> userService.saveOne(user));
复制代码
```

大家可能认为在更多的内核上分配工作执行效率肯定是更快的，然后事实却不总是如此。

## 顺序流

一般情况下，大部分的操作都是用顺序流就行了，使用如下

```java
List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4); 
listOfNumbers.stream().forEach(number -> 
    System.out.println(number + " " + Thread.currentThread().getName()) 
);
复制代码
```

## 并行流

并行流可以让我们在不同的内核上并行执行代码，并将每个单独执行的结果汇总。

但是，执行的顺序是我们无法控制的。每次我们运行程序时它可能会改变。

伪代码：

```java
List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4); listOfNumbers.parallelStream().forEach(number -> 
    System.out.println(number + " " + Thread.currentThread().getName()) 
);
复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90128a5aadf24c2ea9cbc4ae01cd134a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/805760843b074a149215d9823b7e50eb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

这是两次执行的结果，并不相同。

## 代码实例

我们创建一个springboot的应用程序，模拟一个接口请求，循环单个插入500条数据。

先来模拟顺序流：

```java
@PostMapping
public Long saveUser() {
    List<User> users = new ArrayList<>();
    for (int i = 0; i < 500; i++) {
        users.add(new User("小小" + i, "女", 3));
    }
    Long start = System.currentTimeMillis();
    // 为了得到更好的模拟效果（让执行时间拉长），使用单独插入
    users.stream().forEach(user -> userService.saveOne(user));
    Long end = System.currentTimeMillis();
    return end - start;
}
复制代码
```

执行时间为：**44321毫秒**

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91fdc9ff07da4dbeaee82459f5e39f58~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

先来模拟并行流：

```java
@PostMapping
public Long saveUser() {
    List<User> users = new ArrayList<>();
    for (int i = 0; i < 500; i++) {
        users.add(new User("小小" + i, "女", 3));
    }
    Long start = System.currentTimeMillis();
    // 为了得到更好的模拟效果（让执行时间拉长），使用单独插入
    users.parallelStream().forEach(user -> userService.saveOne(user));
    Long end = System.currentTimeMillis();
    return end - start;
}
复制代码
```

执行时间为：**7279毫秒**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a2498206203416cbbd6c776fcdaee03~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

## 总结

可以看到简单的业务处理流程，使用并行流去执行后，效率提升了数倍。下一篇，继续给大家讲解Java 8 并行流的原理和源码解读。