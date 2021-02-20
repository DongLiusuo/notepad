# Callable接口

```java
import java.util.concurrent.Callable;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

class MyThread implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        System.out.println("come in callable");
        return 1024;
    }

}

class CallableDemo {

    public static void main(String[] args) {
        FutureTask<Integer> futureTask = new FutureTask<>(new MyThread());
        Thread t1 = new Thread(futureTask, "a");
        t1.start();
        while (!futureTask.isDone()) {
            // 处理别的
        }
        System.out.println(futureTask.get());//get建议放最后  因为可能会阻塞
    }

}
```

已经有Runnable接口，为什么还要出现Callable接口？请你谈谈它的诞生的前身背景？

