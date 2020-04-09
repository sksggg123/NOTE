# Executor와 Thread의 동작상태 비교

ExecutorService 와 일반 Thread의 차이점은 Executor의 경우 Thread Pool을 활용하여 사용이 되며, Thread는 각기 다른 별도의 Pool이 사용되지 않고 수행이 된다. 아래는 위의 로직 수행 후 결과 값이다.

참고로 **newFixedThreadPool**이 아닌 **newCachedThreadPool()**을 사용할 경우 동적으로 Pool을 생성해준다.

```java
public class MainRest {

    private Integer count = 0;

    public static void main(String[] args) {
        MainRest main = new MainRest();
        main.run();
    }

    public void run() {
        System.out.println("Executor start here...");
        ExecutorService executor = Executors.newFixedThreadPool(2);

        for (int i = 0; i < 10; i++) {
            Test test = new Test();
            executor.execute(test);
            executor.execute(test);
            executor.execute(test);
        }
        executor.shutdown();
        System.out.println("Executor ends here...");

        System.out.println("Thread start here...");
        for (int i = 0; i < 10; i++) {
            Test test = new Test();
            Thread thread1 = new Thread(test);
            Thread thread2 = new Thread(test);
            thread1.start();
            thread2.start();
        }
        System.out.println("Thread ends here...");
    }

    public class Test implements Runnable {
        @Override
        public void run() {
            StringBuilder sb = new StringBuilder();
            sb.append(Thread.currentThread().getName());
            sb.append(Thread.currentThread().getThreadGroup());
            System.out.println(sb.toString() + " [" + ++count + "]");
        }
    }
}
```

```log
Executor start here...
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [1]
Executor ends here...
Thread start here...
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [2]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [3]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [4]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [5]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [6]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [7]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [8]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [9]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [10]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [11]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [12]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [13]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [14]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [15]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [16]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [17]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [18]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [19]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [20]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [21]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [22]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [23]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [24]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [25]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [26]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [27]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [28]
pool-1-thread-1java.lang.ThreadGroup[name=main,maxpri=10] [29]
pool-1-thread-2java.lang.ThreadGroup[name=main,maxpri=10] [30]
Thread-0java.lang.ThreadGroup[name=main,maxpri=10] [31]
Thread-1java.lang.ThreadGroup[name=main,maxpri=10] [32]
Thread-2java.lang.ThreadGroup[name=main,maxpri=10] [33]
Thread-3java.lang.ThreadGroup[name=main,maxpri=10] [34]
Thread-4java.lang.ThreadGroup[name=main,maxpri=10] [35]
Thread-5java.lang.ThreadGroup[name=main,maxpri=10] [36]
Thread-6java.lang.ThreadGroup[name=main,maxpri=10] [37]
Thread-7java.lang.ThreadGroup[name=main,maxpri=10] [38]
Thread-8java.lang.ThreadGroup[name=main,maxpri=10] [39]
Thread-9java.lang.ThreadGroup[name=main,maxpri=10] [40]
Thread-10java.lang.ThreadGroup[name=main,maxpri=10] [41]
Thread-11java.lang.ThreadGroup[name=main,maxpri=10] [42]
Thread-12java.lang.ThreadGroup[name=main,maxpri=10] [43]
Thread-13java.lang.ThreadGroup[name=main,maxpri=10] [44]
Thread-14java.lang.ThreadGroup[name=main,maxpri=10] [45]
Thread-15java.lang.ThreadGroup[name=main,maxpri=10] [46]
Thread-16java.lang.ThreadGroup[name=main,maxpri=10] [47]
Thread-17java.lang.ThreadGroup[name=main,maxpri=10] [48]
Thread ends here...
Thread-19java.lang.ThreadGroup[name=main,maxpri=10] [50]
Thread-18java.lang.ThreadGroup[name=main,maxpri=10] [49]

Process finished with exit code 0
```