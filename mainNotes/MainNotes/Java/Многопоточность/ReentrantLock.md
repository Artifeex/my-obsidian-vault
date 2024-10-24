### Пример использования `ReentrantLock`

Вот пример, иллюстрирующий, как использовать `ReentrantLock` для управления доступом к общему ресурсу:

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Counter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();

    public void increment() {
        lock.lock();  // Захватываем блокировку
        try {
            count++;
        } finally {
            lock.unlock();  // Обязательно освобождаем блокировку в блоке finally
        }
    }

    public int getCount() {
        return count;
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        Thread[] threads = new Thread[10];

        // Создаем и запускаем 10 потоков
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    counter.increment();
                }
            });
            threads[i].start();
        }

        // Ожидаем завершения всех потоков
        for (Thread thread : threads) {
            thread.join();
        }

        System.out.println("Final count: " + counter.getCount());
    }
}
```

### Как это работает

1. **Создание объекта Lock**: В классе `Counter` мы создаем экземпляр `ReentrantLock`, который будет использоваться для управления доступом к переменной `count`.

2. **Захват блокировки**: В методе `increment()` мы захватываем блокировку, вызывая `lock.lock()`. Это позволяет только одному потоку в данный момент времени выполнять код внутри этого блока.

3. **Обработка исключений**: Мы помещаем код, который может вызвать исключение, в блок `try`. Это позволяет нам убедиться, что блокировка будет освобождена даже в случае ошибки.

4. **Освобождение блокировки**: В блоке `finally` мы вызываем `lock.unlock()`, чтобы освободить блокировку и позволить другим потокам доступ к ресурсу.

5. **Запуск потоков**: В методе `main` мы создаем 10 потоков, каждый из которых увеличивает счетчик 1000 раз. После запуска всех потоков основной поток ждет их завершения с помощью `join()`.


