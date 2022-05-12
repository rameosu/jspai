# 一文吃透Fork/Join框架

## 1. 前言

在之前的一篇文章[《巧用并行流，效率快十倍》](/Java/巧用并行流，效率快十倍)中，我简单介绍了Java中并行流 `parallelStream` 的使用，在大部分业务场景下，它可以通过调用更多处理器的内核并行执行流式运算，而且其 `API` 调用极其简单。

Java 中的并行流就是运用了 `Fork/Join` 框架及其公共工作线程池。

今天我们来探究一下其实现原理和 `Fork/Join` 框架的知识。

## 2. Fork/Join 框架

`Fork/Join` 框架最早在 `Java 7` 中出现。它提供了一些工具，通过尝试使用所有可用的`处理器内核`来帮助加速**并行处理**——这是通过**分而治之**的方法来实现的。

分而治之的思想在实践中即分为两步实现：

1. **先分叉（即 forks）- 递归地将任务分解为更小的独立子任务，直到它们简单到可以异步执行。**
2. **后连接（即 join）- 所有子任务的结果递归地连接成单个结果，或者在返回void的任务的情况下，程序简单地等待直到每个子任务被执行。**

### 2.1 拆分源
`Fork/Join` 框架负责在工作线程之间拆分源数据并处理任务完成时的回调。

我们先来看一个并行计算整数和的例子。

```java
public class ParallelStreamDemo {
    public static void main(String[] args) {
        List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4);
        // 串行流
        int sum1 = listOfNumbers.stream().reduce(5, Integer::sum);
        // 并行流
        int sum2 = listOfNumbers.parallelStream().reduce(5, Integer::sum);
        System.out.println("串行流执行结果：" + sum1);
        System.out.println("并行流执行结果：" + sum2);
    }
}
```

执行结果：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/980da7e3652647f4b124086a56c8fc32~tplv-k3u1fbpfcp-watermark.image?)

这里大家是不是会产生疑问：串行流的执行结果符合预期，那为什么并行流的结果会变成预期结果的两倍呢？

这是由于 `reduce` 操作是并行处理的，所以实际上**每个工作线程中的数字 5 都会相加**：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e366fa537db74ba58eaad9d9d1f8b289~tplv-k3u1fbpfcp-watermark.image?)

实际结果可能会因公共 `Fork/Join` 线程池中使用的线程数而异。

为了解决这个问题，应该在并行流之外添加数字 5：

```java
int sum3 = listOfNumbers.parallelStream().reduce(0, Integer::sum) + 5;
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4313cdcb282c49d2bad5032f389b3fb9~tplv-k3u1fbpfcp-watermark.image?)

> 使用并行流时必须需要注意哪些操作可以并行运行。

为了提供有效的并行执行，`Fork/Join` 框架使用称为 `ForkJoinPool` 的线程池，它管理 `ForkJoinWorkerThread` 类型的工作线程。

## 3. ForkJoinPool
`ForkJoinPool` 是 `Fork/Join` 框架的核心。它是 `ExecutorService` 的一个实现，用于管理工作线程并为我们提供工具来获取有关**线程池状态**和**性能的信息**。

工作线程一次只能执行一个任务，但 `ForkJoinPool` 不会为每个子任务创建一个单独的线程，而是将任务存储到池中的每个线程都拥有的`双端队列（deque）`中。

这种线程池的设计对于在 `Work-Stealing Algorithm（工作窃取算法）` 的帮助下平衡线程的工作负载是非常重要的。

### 3.1 Work-Stealing Algorithm
> 工作窃取算法简单地说：空闲线程试图从繁忙线程的双端队列中“窃取”工作。

默认的，一个工作线程是从它自己的 `deque` 的头部任务列表中中获得一个任务。当这个工作线程自己的 `deque` 中的任务列表为空时，该线程会从其他**正在忙**的工作线程 `deque` 的尾部或者**全局入口队列**中获取一个任务。

这种算法可以最大限度地减少了线程竞争任务的可能性。它还减少了线程必须去寻找任务工作的次数，因为它首先处理最大的可用工作块。

### 3.2 ForkJoinPool 实例化
在 `Java 8` 中，访问 `ForkJoinPool` 实例的最便捷方法是使用其静态方法 `commonPool()`。顾名思义，它将提供对公共线程池的引用，这是每个 `ForkJoinTask` 的**默认线程池**。

使用预先定义好的公共线程池可以减少资源消耗，因为我们不再需要为每个任务创建单独的线程池。

```java
ForkJoinPool commonPool = ForkJoinPool.commonPool();
```
通过源码，我们可以看到公共池的创建是在 `ForkJoinPool` 类的静态构造方法中实例化的。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f00d95324b64cbd80efb799fdfa23e6~tplv-k3u1fbpfcp-watermark.image?)

返回了一个 `ForkJoinPool` 类的静态变量 `common`

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba42230be2444d838e1753bbd2cbe6ba~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/454544f1b4d2469eb83c530cf2273f8f~tplv-k3u1fbpfcp-watermark.image?)

在 `ForkJoinPool` 类的 `static{}` 静态方法中，通过调用 `makeCommonPool()` 方法

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f10684dd0494053b7d865db5f8ebd7b~tplv-k3u1fbpfcp-watermark.image?)

最终还是通过 `ForkJoinPool` 的构造器实例化。

在 `Java 7` 中，我们可以在工具类中分配一个公共的静态字段来创建 `ForkJoinPool`。

```java
// 创建一个使用2个处理器内核的线程池，意味着2个线程并行处理
public static ForkJoinPool forkJoinPool = new ForkJoinPool(2);
```

通过源码看一下 `ForkJoinPool` 默认的构造函数。其包含四个方法参数：`并行性`、`线程工厂`、`异常处理器`、`异步模式（先进先出队列/后进先出队列）`。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21031d553c5d4cf5959d061f7dc2104a~tplv-k3u1fbpfcp-watermark.image?)

现在我们可以很方便的访问该线程池：
```java
ForkJoinPool forkJoinPool = PoolUtil.forkJoinPool;
```

## 4. ForkJoinTask
### 4.1 源码讲解
`ForkJoinTask` 是 `ForkJoinPool` 中执行任务的基本类型。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2659d6a347541bdbcd4deed72bd8b44~tplv-k3u1fbpfcp-watermark.image?)

`ForkJoinTask` 是一个抽象类，它两个抽象字类：无返回值的 `RecursiveAction` 和有返回值的 `RecursiveTask<V>`。它们都有一个抽象方法 `compute()`，该方法定义了任务的执行逻辑。

`RecursiveAction` 抽象类源码：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d41d156a14b436eb13f7f6c1cebb26c~tplv-k3u1fbpfcp-watermark.image?)

`RecursiveTask<V>` 抽象类源码：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4afac520e50e4d9f87289a161cf1dc2c~tplv-k3u1fbpfcp-watermark.image?)

### 4.2 自定义 `RecursiveAction` 代码演示

```java
public class CustomRecursiveAction extends RecursiveAction {

    private String workload;

    private static final int THRESHOLD = 4;

    public String getWorkload() {
        return workload;
    }

    public void setWorkload(String workload) {
        this.workload = workload;
    }

    public CustomRecursiveAction(String workload) {
        this.workload = workload;
    }

    /**
     * 为了演示ForkJoin框架的分叉行为，当字符串变量workload的长度大于指定的阈值THRESHOLD时，
     * 使用createSubTask()进行任务拆分
     */
    @Override
    protected void compute() {
        if (workload.length() > THRESHOLD) {
            // 将任务列表提交给ForkJoinTask
            ForkJoinTask.invokeAll(createSubTask());
        } else {
            print(workload);
        }
    }

    /**
     * 递归的创建子任务，子任务将提交给ForkJoinTask，由ForkJoinTask调用重写的compute()
     * @return
     */
    private List<CustomRecursiveAction> createSubTask() {
        List<CustomRecursiveAction> subTasks = new ArrayList<>();

        String partOne = workload.substring(0, workload.length() / 2);
        String partTwo = workload.substring(workload.length() / 2);

        subTasks.add(new CustomRecursiveAction(partOne));
        subTasks.add(new CustomRecursiveAction(partTwo));

        return subTasks;
    }

    /**
     * 打印ForkJoinTask执行结果
     * @param work
     */
    private void print(String work) {
        System.out.println("This result - (" + work + ") - was processed by "
                + Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        // 初始化workload只用4个字符，不会执行分叉逻辑
        CustomRecursiveAction action = new CustomRecursiveAction("abcd");
        action.compute();

        // 第二次执行使用5个字符，执行分叉逻辑
        action.setWorkload("abcde");
        action.compute();
    }
}
```
执行 `main()` 方法，结果如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80ebd84f02194db481503c6fb85c955f~tplv-k3u1fbpfcp-watermark.image?)

如预期，第一次执行 `compute()` 没有分叉，第二次执行，`workload变量`分叉了。

该模式可用于开发您自己的 `RecursiveAction` 类。该类需要：创建一个代表工作总量的对象，选择合适的阈值，定义划分工作的方法，并定义完成工作的方法。

### 4.3 自定义 `RecursiveTask<V>` 代码演示

```java
public class CustomRecursiveTask extends RecursiveTask<String> {

    private String workload;

    private static final int THRESHOLD = 4;

    public String getWorkload() {
        return workload;
    }

    public void setWorkload(String workload) {
        this.workload = workload;
    }

    public CustomRecursiveTask(String workload) {
        this.workload = workload;
    }

    /**
     * 演示ForkJoin框架的分叉行为，当字符串变量workload的长度大于指定的阈值THRESHOLD时，
     * 使用createSubTask()进行任务拆分，通过调用join()触发执行。
     * 使用Java Stream Api 汇总子任务执行的结果
     */
    @Override
    protected String compute() {
        String result;
        if (workload.length() > THRESHOLD) {
            // 将任务列表提交给ForkJoinTask，invokeAll() 方法将子任务提交到公共池并返回一个 Future 列表，调用join()触发连接操作的执行
            result = ForkJoinTask.invokeAll(createSubTask()).stream().map(ForkJoinTask::join).collect(Collectors.joining(""));
        } else {
            result = process(workload);
        }
        System.out.println(result);
        return result;
    }

    /**
     * 递归的创建子任务，子任务将提交给ForkJoinTask，由ForkJoinTask调用重写的compute()
     *
     * @return
     */
    private List<CustomRecursiveTask> createSubTask() {
        List<CustomRecursiveTask> subTasks = new ArrayList<>();

        String partOne = workload.substring(0, workload.length() / 2);
        String partTwo = workload.substring(workload.length() / 2);

        subTasks.add(new CustomRecursiveTask(partOne));
        subTasks.add(new CustomRecursiveTask(partTwo));

        return subTasks;
    }

    /**
     * 处理ForkJoinTask执行结果
     *
     * @param work
     */
    private String process(String work) {
        return work;
    }

    public static void main(String[] args) {
        // 初始化workload只用4个字符，不会执行分叉逻辑
        CustomRecursiveTask action = new CustomRecursiveTask("abcd");
        action.compute();

        // 第二次执行使用5个字符，执行分叉逻辑
        action.setWorkload("abcde");
        action.compute();
    }
}
```

执行 `main()` 方法，结果如下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e5dcb278a7042159a6b3857ef8b6f26~tplv-k3u1fbpfcp-watermark.image?)

如预期，第一次执行 `compute()` 没有分叉，第二次执行，`workload变量`分叉了，并且分别返回了执行的结果和执行 `join()` 后的结果。

## 5. 提交任务到 ForkJoinPool 

可以使用 `submit()` 和 `execute()` 方法将任务提交给线程池。

```java
public class ForkJoinDemo {
    public static void main(String[] args) {
        // 实例化ForkJoinPool
        ForkJoinPool forkJoinPool = ForkJoinPool.commonPool();
        // 实例化有返回值的自定义RecursiveTask
        CustomRecursiveTask customRecursiveTask = new CustomRecursiveTask("abcde");
        // 调用ForkJoinPool的execute方法将任务提交给线程池
        forkJoinPool.execute(customRecursiveTask);
        // 调用ForkJoinTask的join()方法触发连接操作的执行，并获得返回值
        String result = customRecursiveTask.join();
        System.out.println("CustomRecursiveTask result = " + result);

        // 例化无返回值的自定义RecursiveAction
        CustomRecursiveAction customRecursiveAction = new CustomRecursiveAction("abcde");
        // 调用ForkJoinPool的submit方法将任务提交给线程池
        forkJoinPool.submit(customRecursiveAction);
        // 调用ForkJoinTask的join()方法触发连接操作的执行，无返回值
        customRecursiveAction.join();
    }
}
```

您也可以使用 `invoke()` 方法 fork 任务并等待执行结果，不需要手动调用 `join()` 方法来连接。

```java
// 实例化有返回值的自定义RecursiveTask
CustomRecursiveTask customRecursiveTask1 = new CustomRecursiveTask("l拉不拉米");
// 调用ForkJoinPool的invoke方法将任务提交给线程池，并自动执行
String result1 = forkJoinPool.invoke(customRecursiveTask1);
System.out.println("CustomRecursiveTask1 result = " + result1);
```

`invokeAll()` 方法是将 `ForkJoinTasks 集合`提交到 `ForkJoinPool` 的最方便的方法。它将**任务集**作为参数，然后 `fork` 按照它们生成的顺序返回 `Future` 对象的集合。

您也可以分开使用 `fork()` 和 `join()`方法。`fork()`方法将提交一个任务到线程池，但是它不能触发执行连接操作。`join()`方法用于触发执行连接操作。`RecursiveAction` 的 `join()` 方法无返回值； `RecursiveTask` 的 `join()` 方法返回任务执行的结果。

```java
// 例化无返回值的自定义RecursiveAction
CustomRecursiveAction customRecursiveAction1 = new CustomRecursiveAction("l拉不拉米");
// 调用ForkJoinTask的fork方法将任务提交给线程池
customRecursiveAction1.fork();
// 调用ForkJoinTask的join()方法触发连接操作的执行，无返回值
customRecursiveAction1.join();
```
> 最好使用 invokeAll() 方法向 ForkJoinPool 提交多个任务。

## 6. 总结
使用 `Fork/Join` 框架可以加快大型任务的处理速度，但要实现此结果，应遵循一些准则：

- **使用尽可能少的线程池，在大多数情况下，最好的决定是为每个应用程序或系统使用一个线程池**
- **使用默认的公共线程池，除非你需要特殊调优**
- **使用合理的阈值将 ForkJoinTask 拆分为子任务**
- **避免 ForkJoinTasks 中的任何阻塞**

演示源码在 [GitHub](https://github.com/rameosu/ForkJoin)