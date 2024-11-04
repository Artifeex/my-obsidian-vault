Да, для задач, аннотированных `@Async`, Spring использует отдельный пул потоков, отличающийся от того, что используется для `@Scheduled`. Это значит, что Spring действительно использует несколько различных пулов потоков для различных типов задач, чтобы управлять параллельным выполнением и избежать взаимного блокирования ресурсов.

### Основные пулы потоков, используемые Spring

Spring по умолчанию может использовать несколько разных пулов потоков для различных задач:

1. **`@Async`** — использует **`TaskExecutor`**, обычно на базе `SimpleAsyncTaskExecutor` или `ThreadPoolTaskExecutor`. Этот пул применяется для асинхронных задач, то есть для методов, аннотированных `@Async`. По умолчанию пул имеет небольшой размер, но его можно настроить для поддержки многопоточного выполнения асинхронных задач.

2. **`@Scheduled`** — использует **`ScheduledThreadPoolExecutor`**, который применяется для задач, аннотированных `@Scheduled`. Этот пул по умолчанию имеет один поток, но его также можно настроить через `ThreadPoolTaskScheduler` для увеличения количества потоков, если нужно обрабатывать несколько задач одновременно.

3. **Web-запросы (если используется Spring MVC)** — использует **`Web server thread pool`** (например, Tomcat или Jetty), который обрабатывает входящие HTTP-запросы. Этот пул обычно управляется контейнером сервлетов и не связан напрямую с `@Async` или `@Scheduled`. Однако при наличии асинхронных запросов или запросов с долгими операциями можно также использовать `@Async` для их обработки в другом пуле потоков.

### Настройка пула потоков для `@Async`

Для кастомизации пула потоков, используемого `@Async`, можно определить `AsyncConfigurer` и настроить свой `ThreadPoolTaskExecutor`:

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@Configuration
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);     // Основное количество потоков
        executor.setMaxPoolSize(10);     // Максимальное количество потоков
        executor.setQueueCapacity(25);   // Очередь задач, если все потоки заняты
        executor.setThreadNamePrefix("AsyncTask-");
        executor.initialize();
        return executor;
    }
}
```

Теперь все методы, аннотированные `@Async`, будут использовать этот кастомный `ThreadPoolTaskExecutor`.

### Подводя итоги

Таким образом, Spring может использовать до трех (или более) отдельных пулов потоков:

- **Для `@Scheduled`** — `ScheduledThreadPoolExecutor` (с возможностью настройки через `ThreadPoolTaskScheduler`).
- **Для `@Async`** — `ThreadPoolTaskExecutor` (настраиваемый через `AsyncConfigurer`).
- **Для обработки HTTP-запросов** — пул потоков веб-сервера (Tomcat, Jetty или другой сервер).

Каждый из этих пулов работает независимо, что позволяет эффективно разделять нагрузку и управлять потоками для разных типов задач.