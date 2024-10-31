В Spring есть несколько типов автоматического связывания (или инъекции зависимостей), которые позволяют эффективно управлять зависимостями между бинами. Основные методы автоматического связывания включают:

### 1. **Автоматическое связывание по типу** (by type)

При использовании автоматического связывания по типу Spring ищет бин, соответствующий типу параметра конструктора или полю. Если в контейнере есть именно один подходящий бин, Spring инжектирует его. Если их несколько, возникает исключение.

**Пример**:
```java
@Component
public class ServiceA {
}

@Component
public class ServiceB {
    private final ServiceA serviceA;

    @Autowired
    public ServiceB(ServiceA serviceA) { // связывание по типу
        this.serviceA = serviceA;
    }
}
```

### 2. **Автоматическое связывание по имени** (by name)

Если автоматическое связывание по типу не находит уникальный бин, можно использовать связывание по имени. При этом Spring ищет бин, имя которого совпадает с именем параметра конструктора или поля.

**Пример**:
```java
@Component
public class ServiceA {
}

@Component
public class ServiceB {
    private final ServiceA serviceA;

    @Autowired
    public ServiceB(@Qualifier("serviceA") ServiceA serviceA) { // связывание по имени
        this.serviceA = serviceA;
    }
}
```

### 3. **Использование аннотации `@Autowired`**

Аннотация `@Autowired` указывает Spring автоматически инжектировать зависимость. Она может применяться к полям, конструкторам и методам.

#### a. **На полях**
```java
@Component
public class ServiceB {
    @Autowired
    private ServiceA serviceA; // связывание по типу
}
```

#### b. **На конструкторах**
```java
@Component
public class ServiceB {
    private final ServiceA serviceA;

    @Autowired
    public ServiceB(ServiceA serviceA) { // связывание по типу
        this.serviceA = serviceA;
    }
}
```

#### c. **На методах**
```java
@Component
public class ServiceB {
    private ServiceA serviceA;

    @Autowired
    public void setServiceA(ServiceA serviceA) { // связывание по типу
        this.serviceA = serviceA;
    }
}
```

### 4. **Использование аннотации `@Qualifier`**

Аннотация `@Qualifier` используется в сочетании с `@Autowired`, чтобы указать, какой именно бин инжектировать, если в контейнере есть несколько подходящих кандидатов. Это позволяет избежать исключений при неуникальных зависимостях.

**Пример**:
```java
@Component
public class ServiceA {
}

@Component
public class ServiceB {
    private final ServiceA serviceA;

    @Autowired
    public ServiceB(@Qualifier("serviceA") ServiceA serviceA) { // указываем конкретный бин
        this.serviceA = serviceA;
    }
}
```

### 5. **Автоматическое связывание по типу коллекций**

Если в контейнере есть несколько бинов одного типа, они могут быть инжектированы как коллекция. Spring автоматически инжектирует все соответствующие бины в `List`, `Set` или `Map`.

**Пример**:
```java
@Component
public class ServiceA {
}

@Component
public class ServiceB {
    private final List<ServiceA> services; // связывание коллекции

    @Autowired
    public ServiceB(List<ServiceA> services) { // автоматически инжектируются все ServiceA
        this.services = services;
    }
}
```

### 6. **Внедрение значений свойств**

Инъекция значений из файлов конфигурации (например, `application.properties`) может быть выполнена с использованием аннотации `@Value`.

**Пример**:
```java
@Component
public class MyComponent {
    @Value("${my.property}") // связывание значения из конфигурации
    private String myProperty;
}
```

### 7. **Использование аннотации `@Resource`**

Аннотация `@Resource` является частью JSR-250 и может использоваться для связывания по имени. Она работает аналогично `@Autowired` и `@Qualifier`, но приоритет отдается связыванию по имени.

**Пример**:
```java
@Component
public class ServiceB {
    @Resource // связывание по имени
    private ServiceA serviceA;
}
```

### Выводы

- **Автоматическое связывание** — это мощный механизм, позволяющий упростить управление зависимостями в Spring-приложениях.
- **`@Autowired`** и **`@Qualifier`** предоставляют гибкость при выборе зависимостей.
- **Инъекция коллекций** позволяет легко управлять группами бинов одного типа.
- **`@Value`** и **`@Resource`** обеспечивают дополнительные способы конфигурации и управления зависимостями. 

Эти методы автоматического связывания помогают строить гибкие и поддерживаемые приложения, уменьшая объем необходимого кода для управления зависимостями.