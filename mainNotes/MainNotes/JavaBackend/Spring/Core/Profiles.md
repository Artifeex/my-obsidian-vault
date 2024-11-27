### Профили (Profiles) в Spring

**Профили** в Spring — это механизм для определения различных конфигураций приложения в зависимости от среды выполнения (например, **разработка**, **тестирование**, **продакшен**). Это позволяет гибко переключаться между наборами настроек, не изменяя код.

---

### Когда использовать профили?

1. **Разные окружения:**
    - Например, локальная разработка, CI/CD тестирование, staging или production.
    - Различные базы данных, API-ключи, порты, кэш-конфигурации и т.д.
2. **Разные сервисы:**
    - Использование разных реализаций интерфейсов (например, заглушки/mock-сервисы в тестах и реальные сервисы в продакшене).
3. **Изоляция конфигураций:**
    - Вы хотите содержать конфигурации, специфичные для среды, отдельно друг от друга для удобства управления.

---

### Как настроить и работать с профилями в Spring

#### 1. **Определение профилей**

Spring профили определяются с помощью аннотации `@Profile` на уровне классов или бинов.

Пример:

```java
@Profile("dev")
@Configuration
public class DevConfig {
    @Bean
    public DataSource dataSource() {
        return new HikariDataSource(); // DataSource для dev
    }
}
```

```java
@Profile("prod")
@Configuration
public class ProdConfig {
    @Bean
    public DataSource dataSource() {
        return new SomeProductionDataSource(); // DataSource для prod
    }
}
```

#### 2. **Активизация профилей**

Профиль можно активировать разными способами:

- **Через переменную окружения:**
    
    ```bash
    export SPRING_PROFILES_ACTIVE=dev
    ```
    
- **Через параметры JVM:**
    
    ```bash
    -Dspring.profiles.active=dev
    ```
    
- **В application.properties:**
    
    ```properties
    spring.profiles.active=dev
    ```
    
- **Через код (не рекомендуется):**
    
    ```java
    ConfigurableEnvironment environment = context.getEnvironment();
    environment.setActiveProfiles("dev");
    ```
    

---

#### 3. **Файлы конфигурации для профилей**

Spring поддерживает профили в конфигурационных файлах, таких как `application.properties` или `application.yml`. Для профилей создаются отдельные файлы с названием `application-{profile}.properties` или `application-{profile}.yml`.

##### Пример:

`application.properties` (общие настройки):

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=
```

`application-dev.properties` (для dev):

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/devdb
spring.datasource.username=devuser
spring.datasource.password=devpass
```

`application-prod.properties` (для prod):

```properties
spring.datasource.url=jdbc:mysql://prod-url:3306/proddb
spring.datasource.username=produser
spring.datasource.password=prodpass
```

Spring автоматически загрузит конфигурацию из файла, соответствующего активному профилю.

---

#### 4. **Комбинирование профилей**

Иногда требуется использовать сразу несколько профилей:

- **Профиль по умолчанию (`default`):** Если ни один профиль не активен, используется профиль `default`.
    
    ```java
    @Profile("default")
    @Bean
    public MyBean defaultBean() {
        return new MyBean("Default");
    }
    ```
    
- **Множественные активные профили:** Укажите несколько профилей через запятую:
    
    ```properties
    spring.profiles.active=dev,feature-x
    ```
    
    В этом случае Spring загружает настройки обоих профилей.
    

---

#### 5. **Профили в тестировании**

Для тестов можно явно указать, какой профиль использовать, с помощью `@ActiveProfiles`:

```java
@ActiveProfiles("test")
@SpringBootTest
public class MyServiceTest {
    // Тесты с активным профилем "test"
}
```

---

### Лучшая практика работы с профилями

1. **Четкое разделение окружений:**
    - Держите настройки для разработки, тестирования и продакшена раздельно.
2. **Используйте профиль `default` для общих настроек.**
3. **Не храните секреты в файлах конфигурации:**
    - Используйте менеджеры секретов (например, HashiCorp Vault или AWS Secrets Manager).
4. **Минимизируйте использование явного кода для активации профилей:**
    - Настройки среды и файлы конфигурации более гибкие и надежные.
5. **Проверяйте профили в тестах.**
6. **Документируйте профили, чтобы разработчики знали их предназначение.**

---

### Часто используемые кейсы

#### 1. **Разные базы данных:**

```java
@Profile("dev")
@Bean
public DataSource h2DataSource() {
    return new HikariDataSource();
}

@Profile("prod")
@Bean
public DataSource mysqlDataSource() {
    return new MysqlDataSource();
}
```

#### 2. **Логирование для разных профилей:**

В `application-dev.properties`:

```properties
logging.level.root=DEBUG
```

В `application-prod.properties`:

```properties
logging.level.root=ERROR
```

#### 3. **Заглушки для сервисов:**

```java
@Profile("test")
@Bean
public MyService mockService() {
    return Mockito.mock(MyService.class);
}

@Profile("prod")
@Bean
public MyService realService() {
    return new MyServiceImpl();
}
```

---

### Вывод

Spring Profiles — это мощный инструмент для управления конфигурациями в разных средах. Используйте их для:

- Изоляции настроек,
- Подключения заглушек в тестах,
- Переключения между конфигурациями для разработки и продакшена.

Работа с профилями помогает сделать код более гибким, удобным для сопровождения и адаптируемым под разные условия выполнения.