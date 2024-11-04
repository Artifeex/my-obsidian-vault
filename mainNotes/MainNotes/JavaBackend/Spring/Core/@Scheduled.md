Да, по умолчанию Spring использует `ScheduledThreadPool`, который является частью планировщика задач на основе пула потоков (`ScheduledThreadPoolExecutor`). Этот пул потоков позволяет выполнять несколько запланированных задач параллельно, что предотвращает их блокировку, если одна из задач выполняется долго.

По умолчанию `ScheduledThreadPool` имеет размер пула в один поток. Однако это значение можно изменить, если вам нужно выполнять несколько задач одновременно. 

### Настройка пула потоков для задач планировщика

Чтобы изменить размер пула, можно определить кастомный `TaskScheduler`, например, с использованием `ThreadPoolTaskScheduler`:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;

@Configuration
public class SchedulerConfig {

    @Bean
    public ThreadPoolTaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(5);  // Устанавливаем нужный размер пула
        scheduler.setThreadNamePrefix("ScheduledTask-");
        return scheduler;
    }
}
```

### Как это работает

В данном примере `ThreadPoolTaskScheduler` с пулом из 5 потоков будет назначен в качестве планировщика для всех задач, аннотированных `@Scheduled`. Это позволяет запускать до 5 задач одновременно. Если пул потоков меньше, чем количество одновременно запланированных задач, задачи будут ожидать, пока освободится поток.

### Использование `@Async` для дополнительной параллельности

Если требуется, чтобы определенные задачи выполнялись асинхронно, можно дополнительно использовать аннотацию `@Async`.