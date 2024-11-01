Событийный подход в Spring основан на использовании механизма **публикации и обработки событий**. Этот подход позволяет модулям приложения обмениваться информацией, не зная друг о друге напрямую, что способствует **ослаблению связи** между компонентами и упрощает поддержку и расширение системы. 

### Основные компоненты событийного подхода

1. **События (`Events`)**: Это объекты, представляющие состояние или действие, о которых необходимо уведомить другие компоненты. Обычно события создаются как простые классы, содержащие информацию, которую нужно передать.
   
2. **Издатели событий (`Event Publishers`)**: Компоненты, которые создают и публикуют события, передавая их другим частям системы через специальный сервис, например, `ApplicationEventPublisher`.

3. **Слушатели событий (`Event Listeners`)**: Компоненты, которые подписаны на определенные события и выполняют определенные действия, когда это событие происходит. В Spring слушатели могут быть аннотированы `@EventListener`, и для транзакционных событий — `@TransactionalEventListener`.

### Как работает событийный подход в Spring

1. **Создание события**:
   Событие — это простой Java-класс, наследуемый от `ApplicationEvent` (хотя это не обязательно). Например:

    ```java
    public class UserDeletedEvent {
        private final Long userId;
        private final String username;

        public UserDeletedEvent(Long userId, String username) {
            this.userId = userId;
            this.username = username;
        }

        public Long getUserId() { return userId; }
        public String getUsername() { return username; }
    }
    ```

2. **Публикация события**:
   События публикуются с помощью `ApplicationEventPublisher`. Для этого компонент, который вызывает событие, внедряет `ApplicationEventPublisher` и вызывает метод `publishEvent`:

    ```java
    @Service
    public class UserService {

        private final ApplicationEventPublisher eventPublisher;

        public UserService(ApplicationEventPublisher eventPublisher) {
            this.eventPublisher = eventPublisher;
        }

        public void deleteUser(Long userId) {
            // Выполняем удаление пользователя
            User user = userRepository.findById(userId).orElseThrow();
            userRepository.delete(user);

            // Публикуем событие
            eventPublisher.publishEvent(new UserDeletedEvent(userId, user.getUsername()));
        }
    }
    ```

3. **Обработка событий слушателем**:
   Чтобы реагировать на событие, создается компонент-слушатель, который выполняет необходимую логику в ответ на событие. Для этого используется аннотация `@EventListener`:

    ```java
    @Component
    public class UserEventListener {

        @EventListener
        public void handleUserDeletedEvent(UserDeletedEvent event) {
            System.out.println("User with ID " + event.getUserId() + " was deleted");
            // Дополнительная логика обработки
        }
    }
    ```

   > `@EventListener` позволяет слушателю подписаться на событие и автоматически обрабатывать его при возникновении.

4. **Обработка событий в контексте транзакции**:
   Если нужно обработать событие только после завершения транзакции, используется аннотация `@TransactionalEventListener`. Этот тип событий вызывается **только после успешного коммита транзакции** (например, когда данные точно удалены из базы):

    ```java
    @Component
    public class UserEventListener {

        @TransactionalEventListener
        public void handleUserDeletedEvent(UserDeletedEvent event) {
            System.out.println("Transactional event for user ID " + event.getUserId() + " was committed");
            // Логика обработки, например, очистка кэша
        }
    }
    ```

   У `@TransactionalEventListener` есть несколько режимов запуска:
   - `AFTER_COMMIT` (по умолчанию): срабатывает после успешного завершения транзакции.
   - `AFTER_ROLLBACK`: срабатывает после отката транзакции.
   - `AFTER_COMPLETION`: срабатывает после завершения транзакции вне зависимости от ее состояния.
   - `BEFORE_COMMIT`: срабатывает перед завершением транзакции.

### Преимущества событийного подхода

1. **Ослабление связей**: Издатель не зависит от конкретных слушателей. Это упрощает тестирование и добавление новых функций без изменения основного кода.

2. **Асинхронная обработка**: События могут обрабатываться асинхронно, что позволяет разгрузить основной поток. Для этого можно добавить аннотацию `@Async` над методом слушателя.

    ```java
    @Async
    @EventListener
    public void handleUserDeletedEvent(UserDeletedEvent event) {
        // Асинхронная обработка события
    }
    ```

3. **Удобство для задач с отложенной обработкой**: Использование событийного подхода хорошо подходит для задач, которые нужно выполнить после основного действия, например, отправки уведомлений, обновления кэша, ведения логов и т.д.

### Пример использования событийного подхода в приложении

Представьте, что после удаления пользователя из базы данных необходимо:
- Очистить его данные из кэша.
- Отправить уведомление администратору.
- Записать удаление в журнал.

Вместо того, чтобы помещать весь этот код в метод удаления, можно просто опубликовать `UserDeletedEvent` и создать три независимых слушателя для этих действий, что упростит поддержку и тестирование кода:

```java
@EventListener
public void clearUserCache(UserDeletedEvent event) {
    // Логика очистки кэша
}

@EventListener
public void notifyAdmin(UserDeletedEvent event) {
    // Логика отправки уведомления
}

@EventListener
public void logUserDeletion(UserDeletedEvent event) {
    // Логика записи в журнал
}
```

### Итог

Событийный подход в Spring помогает улучшить архитектуру приложения за счет разделения логики, асинхронной обработки и удобной организации действий, которые происходят после основных операций, делая приложение более гибким и масштабируемым.