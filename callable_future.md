### Intro
2 cách để tạo một Thread trong Java: tạo 1 đối tượng của lớp được extend từ class `Thread` hoặc implements từ `interface` Runnable. Trong bài viết này tôi giới thiệu với các bạn một cách khác để tạo Thread, đó là `Callable` trong Java với khả năng trả về kết quả `Future<T>` sau khi xử lý và có thể throw Exception nếu trong quá trình xử lý tác vụ có lỗi xảy ra.

#### Callable
Callable là một interface sử dụng Java Generic để định nghĩa đối tượng sẽ trả về sau khi xử lý xong tác vụ.

Callable interface cũng giống như Runnable, ngoại trừ khác ở chỗ thay vì trả về void từ run() method của Runnable thì call() method của Callable trả về đối tượng kiểu Future<T> (bất kỳ) hoặc throw Exception.

Runnable:

```
public abstract void run() => kiểu trả về là void
```

Callable:

```
<T> Future<T> submit(Callable<T> task) => kiểu trả về là Future<T>

<T> Future<T> submit(Runnable<T> task) => kiểu trả về là Future<T>
```

#### Future
Dùng để lấy kết quả khi thực thi một Thread, có thể lấy Future khi submit một task vào ThreadPool. Task ở đây là một object implement Runnable hay Callable.

Một số method của Future:

- `isDone()` : Kiểm tra task hoàn thành hay không?
- `cancel()` : Hủy task
- `isCancelled()` : Kiểm tra task bị hủy trước khi hoàn thành hay không?
- `get()` : Lấy kết quả trả về của task.

### Ví dụ sử dụng Callable và Future

__Sử dụng phương thức submit(Callable) của ExecutorService với kiểu trả về là 1 Future<T>__

Ví dụ về tính tổng bình phương của 10 số, thay vì xử lý việc tính tổng này trong Thread chính của chương trình, tôi sẽ tạo mới Thread sử dụng Callable để xử lý và nhận kết quả về thông qua Future.

Tạo Callable worker

```java
package com.gpcoder.threadpool.callable;
 
import java.util.Random;
import java.util.concurrent.Callable;
 
public class CallableWorker implements Callable<Integer> {
 
    private int num;
    private Random ran;
 
    public CallableWorker(int num) {
        this.num = num;
        this.ran = new Random();
    }
 
    public Integer call() throws Exception {
        Thread.sleep(ran.nextInt(10) * 1000);
        int result = num * num;
        System.out.println("Complete " + num);
        return result;
    }
}
```

Để thực thi tác vụ của Callable, chúng ta phải submit nó vào một `ThreadPool` sử dụng phương thức `submit()` của Executor Framework. Để nhận kết quả trả về chúng ta sử dụng phương thức `get()` của lớp Future. Ta có chương trình như bên dưới:

```java
package com.gpcoder.threadpool.callable;
 
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
 
public class CallableExample {
    public static void main(String[] args) {
        // create a list to hold the Future object associated with Callable
        List<Future<Integer>> list = new ArrayList<>();
 
        // Get ExecutorService from Executors utility class, thread pool size is 5
        ExecutorService executor = Executors.newFixedThreadPool(5);
 
        Callable<Integer> callable;
        Future<Integer> future;
        for (int i = 1; i <= 10; i++) {
            callable = new CallableWorker(i);
 
            // submit Callable tasks to be executed by thread pool
            future = executor.submit(callable);
 
            // add Future to the list, we can get return value using Future
            list.add(future);
        }
 
        // shut down the executor service now
        executor.shutdown();
 
        // Wait until all threads are finish
        while (!executor.isTerminated()) {
            // Running ...
        }
 
        int sum = 0;
        for (Future<Integer> f : list) {
            try {
                sum += f.get();
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }
 
        System.out.println("Finished all threads: ");
        System.out.println("Sum all = " + sum);
    }
}
```

Thực thi chương trình trên, ta có kết quả như sau:

```
Complete 5
Complete 4
Complete 7
Complete 6
Complete 3
Complete 2
Complete 9
Complete 1
Complete 8
Complete 10
Finished all threads: 
Sum all = 385
```
Lưu ý: Tương tự như submit(Callable), phương thức submit(Runnable) cũng đưa vào 1 Runnable và nó trả về một đối tượng Future. Đối tượng Future có thể được sử dụng để kiểm tra nếu Runnable đã hoàn tất việc thực thi.

__Sử dụng phương thức get() của Future<T> với Timeout__

Phương thức get() là synchronous, do đó nó sẽ blocking chương trình của chúng ta mỗi khi đợi kết quả của Callable. Để hạn chế blocking chương trình quá lâu, chúng ta có thể sử dụng phương thức get() này với một thời gian Timeout như sau:

```java
future.get(7, TimeUnit.SECONDS);
```

Lưu ý: khi sử dụng phương thức get() với Timeout có thể nhận một TimeoutException nếu thời gian thực thi của task vượt quá khoảng thời gian Timeout.

