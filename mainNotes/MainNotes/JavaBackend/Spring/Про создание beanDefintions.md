##### Xml конфигурация

Для Xml конфигурации используется класс — _XmlBeanDefinitionReader_, который реализует интерфейс _BeanDefinitionReader_. Тут все достаточно прозрачно. _XmlBeanDefinitionReader_ получает _InputStream_ и загружает _Document_ через _DefaultDocumentLoader_. Далее обрабатывается каждый элемент документа и если он является бином, то создается _BeanDefinition_ на основе заполненных данных (id, name, class, alias, init-method, destroy-method и др.). Каждый _BeanDefinition_ помещается в Map. Map хранится в классе _DefaultListableBeanFactory_. В коде Map выглядит вот так.  
  

```java
/** Map of bean definition objects, keyed by bean name */private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(64);
```

  

##### Конфигурация через аннотации с указанием пакета для сканирования или JavaConfig

  
Конфигурация через аннотации с указанием пакета для сканирования или JavaConfig в корне отличается от конфигурации через xml. В обоих случаях используется класс _AnnotationConfigApplicationContext_.  
  

```java
new AnnotationConfigApplicationContext(JavaConfig.class);
```

  
или  
  

```java
new AnnotationConfigApplicationContext(“package.name”);
```

  
Если заглянуть во внутрь AnnotationConfigApplicationContext, то можно увидеть два поля.  
  

```java
private final AnnotatedBeanDefinitionReader reader;private final ClassPathBeanDefinitionScanner scanner;
```

  
_ClassPathBeanDefinitionScanner_ сканирует указанный пакет на наличие классов помеченных аннотацией _[Component](https://habr.com/ru/users/component/)_ (или любой другой аннотацией которая включает в себя _[Component](https://habr.com/ru/users/component/)_). Найденные классы парсируются и для них создаются _BeanDefinition_.  
Чтобы сканирование было запущено, в конфигурации должен быть указан пакет для сканирования.  
  

```java
@ComponentScan({"package.name"})
```

  
или  
  

```java
<context:component-scan base-package="package.name"/>
```

  
_AnnotatedBeanDefinitionReader_ работает в несколько этапов.  

1. Первый этап — это регистрация всех _@Configuration_ для дальнейшего парсирования. Если в конфигурации используются _Conditional_, то будут зарегистрированы только те конфигурации, для которых _Condition_ вернет true. Аннотация _Conditional_ появилась в четвертой версии спринга. Она используется в случае, когда на момент поднятия контекста нужно решить, создавать бин/конфигурацию или нет. Причем решение принимает специальный класс, который обязан реализовать интерфейс _Condition_.
2. Второй этап — это регистрация специального _BeanFactoryPostProcessor_, а именно _BeanDefinitionRegistryPostProcessor_, который при помощи класса _ConfigurationClassParser_ парсирует JavaConfig и создает _BeanDefinition_.

1. Срабатывает статический блок инициализации
2. Срабатывает не статический блок инициализации
3. Внедрение зависимостей на основе конструктора
4. Внедрение зависимостей на основе `setter`-ов
5. **

1. Отрабатывают методы стандартного набора `Aware` интерфейсов
    
2. `BeanPostProcessor#postProcessBeforeInitialization` обрабатывает bean-компонент
    
3. `InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization` вызывает методы обратного вызова, помеченные аннотацией `@PostConstruct`
    
4. `BeanFactory` вызывает метод `InitializingBean#afterPropertiesSet`
    
5. `BeanFactory` вызывает метод обратного вызова, зарегистрированный как `initMethod`
    
6. `BeanPostProcessor#postProcessAfterInitialization` обрабатывает bean-компонент
    
