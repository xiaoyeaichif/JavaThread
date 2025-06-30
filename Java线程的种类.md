# Java线程的分类

## 1.线程的概述

- Java 线程表面上是由 JVM 管理，实际上底层执行依赖的是操作系统的线程机制。JVM 只是桥梁，它把 Java 代码的线程意图转换为 OS 可以理解并调度的线程操作。

- Java Thread（你写的 Thread）
            │
            ▼
    JVM 调度管理
            │
            ▼
    操作系统原生线程（pthread, CreateThread）
            │
            ▼
    CPU 调度器 负责实际运行


## 2.Java 线程的种类

### 2.1 创建线程的方式一： 继承Thread类

    1. 创建一个继承Thread类的子类
    2. 重写Thread类中的run()方法，将此线程要执行的操作声明在此方法中
    3. 创建当前Thread的子类对象
    4. 通过对象调用start()方法启动此线程

- 示例
  
``` Java
public class studyThread {
    public static void main(String[] args) {
        // 3:创建线程对象
        MyThread thread1 = new MyThread();
        // 4：启动线程
        // 需要注意start()方法只能调用一次，不然会出现异常；要想重新启动线程，需要创建新的对象
        thread1.start();

        // 创建第二个对象
        MyThread thread2 = new MyThread();
        thread2.start();

        // 主线程执行的操作
        System.out.println("主线程的执行");
        for (int i = 0; i <= 100; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() +":"+"i = " + i);
            }
        }
    }
}

// 1：MyThread继承Thread
class MyThread extends Thread {
    @Override
    // 2：重写run方法
    public void run() {
        // 打印100以内的偶数
        for (int i = 0; i <= 100; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() +":"+"i = " + i);
            }
        }
    }
}
```

Java创建线程执行的流程如下：
Java中调用start() ─────► JVM调用start0() ─────► 调用OS创建线程（如pthread_create）
                                            │
                                            |
                                            ▼
                                   系统线程启动后回调 Thread.run()



### 2.2 创建线程的方式二： 实现Runnable接口

    1.定义一个实现Runnable接口的类
    2.实现类实现Runnable接口中的run方法，写上要执行的任务，并将任务代码写在run方法中
    3.创建实现类对象
    4.创建Thread类对象，构造方法中传递Runnable接口实现类对象
    5.调用Thread类中的start方法，启动线程

- 示例
  
```Java
public class RunableUse {
    public static void main(String[] args) {
        // 3：创建类的对象
        EvenRunnable evenRunnable = new EvenRunnable();
        // 4：创建线程对象
        Thread thread1 = new Thread(evenRunnable);
        // 5：启动线程，调用start方法
        thread1.start();


        // 主线程实现的方法
        for(int i = 1;i <= 100;i++){
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + ": i = " + i);
            }
        }

        // 匿名方式创建线程
        new Thread(new Runnable() {
            // 重写run方法
            public void run() {
                for (int i = 1; i <= 100; i++) {
                    if (i % 2 == 0) {
                        System.out.println(Thread.currentThread().getName() + ": i = " + i);
                    }
                }
            }
        }).start();
    }
}


// 1：创建Runable接口的实现
// 实现Runnable接口
class EvenRunnable implements Runnable {
    @Override
    public void run() {
        // 打印100以内的偶数
        for (int i = 1; i <= 100; i++) {
            if (i % 2 == 0) {
                System.out.println(Thread.currentThread().getName() + ": i = " + i);
            }
        }
    }
}
```


### 2.3 实现 Callable 接口 + FutureTask（支持返回值和异常）(和C++的future、promise类似)

```java
class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        // 线程执行任务
        return 123;
    }
}

public class Demo {
    public static void main(String[] args) throws Exception {
        FutureTask<Integer> task = new FutureTask<>(new MyCallable());
        Thread t = new Thread(task);
        t.start();

        // 阻塞等待结果
        Integer result = task.get();
        System.out.println("线程返回值: " + result);
    }
}
```

### 2.4 使用线程池（ExecutorService）

```Java
public class Demo {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(3);

        pool.execute(() -> System.out.println("线程池执行任务"));

        pool.shutdown();
    }
}
```

### 2.5 Java 19+ 虚拟线程（Preview）

- 虚拟线程允许程序员创建线程，这些线程与Java线程一样，可以执行任务，但是它们不占用JVM的线程资源。

```Java
Thread.startVirtualThread(() -> System.out.println("虚拟线程执行"));
```

- 不同方式创建线程的对比

| 方式                       | 是否继承 Thread | 是否有返回值  | 适用场景           |
| ------------------------ | ----------- | ------- | -------------- |
| 继承 Thread                | 是           | 否       | 简单线程任务         |
| 实现 Runnable              | 否           | 否       | 线程任务复用，多线程共享资源 |
| 实现 Callable + FutureTask | 否           | 是       | 需要返回值的线程任务     |
| 线程池 ExecutorService      | 否           | 视提交任务而定 | 线程管理与复用        |
| 虚拟线程                     | 否           | 视任务而定   | 大量并发任务，轻量级线程   |
