ReentrantReadWriteLock - это реализация интерфейса ReadWriteLock. Под капотом на самом деле есть два поля типа [[ReentrantLock]] - один для читателей, второй для писателей.
### Пример использования `ReentrantReadWriteLock`

Если необходимо обеспечить одновременный доступ для читателей и эксклюзивный доступ для писателей, можно использовать `ReentrantReadWriteLock`:

```java
import java.util.concurrent.locks.ReentrantReadWriteLock;

class SharedResource {
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private int data;

    public void write(int value) {
        rwLock.writeLock().lock();  // Захватываем писательскую блокировку
        try {
            data = value;
        } finally {
            rwLock.writeLock().unlock();  // Освобождаем писательскую блокировку
        }
    }

    public int read() {
        rwLock.readLock().lock();  // Захватываем читательскую блокировку
        try {
            return data;
        } finally {
            rwLock.readLock().unlock();  // Освобождаем читательскую блокировку
        }
    }
}

public class Main {
    public static void main(String[] args) {
        SharedResource resource = new SharedResource();

        // Создание потоков для записи
        Thread writer = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                resource.write(i);
                System.out.println("Записано: " + i);
            }
        });

        // Создание потоков для чтения
        Thread reader = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                int value = resource.read();
                System.out.println("Прочитано: " + value);
            }
        });

        writer.start();
        reader.start();
    }
}
```

### Как это работает

1. **Читательская и писательская блокировка**: В классе `SharedResource` мы создаем экземпляр `ReentrantReadWriteLock`, который предоставляет два типа блокировок: `readLock()` и `writeLock()`.

2. **Запись данных**: В методе `write()` мы захватываем писательскую блокировку. Это предотвращает доступ других потоков (как читателей, так и писателей) к ресурсу, пока идет запись.

3. **Чтение данных**: В методе `read()` мы захватываем читательскую блокировку. Несколько потоков могут захватывать читательскую блокировку одновременно, но при этом запись будет заблокирована.