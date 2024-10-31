
Этап инициализации bean-компонента
1. `BeanFactory` создает bean-компонент

Парсим конфигурацию (xml, JavaConfig, через аннотации, …) создаем BeanDefinitions - это специальный интерфейс, через который можно получить доступ к метаданным будущего бина.

- Для xml конфигурации: XmlBeanDefinitionReader, который реализует интерфейс BeanDefinitionReader. Тут все достаточно прозрачно. XmlBeanDefinitionReader получает InputStream и загружает Document через DefaultDocumentLoader. Далее обрабатывается каждый элемент документа и если он является бином, то создается BeanDefinition на основе заполненных данных (id, name, class, alias, init-method, destroy-method и др.). Каждый BeanDefinition помещается в Map. Map хранится в классе DefaultListableBeanFactory.
    
- Для JavaConfig или через аннотации: используется класс
    

|   |
|---|
|new AnnotationConfigApplicationContext(JavaConfig.class); // или имя пакета|

Этот класс содержит

- private final ClassPathBeanDefinitionScanner reader; сканирует и ищет @Component. Найденные классы парсируются и для них создаются BeanDefinition.
    
- private final AnnotatedBeanDefinitionReader scanner; регистрация всех @Configuration для дальнейшего парсирования. Если в конфигурации используются Conditional, то будут зарегистрированы только те конфигурации, для которых Condition вернет true. 
    

  

Второй этап — это регистрация специального BeanFactoryPostProcessor, а именно BeanDefinitionRegistryPostProcessor, который при помощи класса ConfigurationClassParser парсирует JavaConfig и создает BeanDefinition.

2. Срабатывает статический блок инициализации
    
3. Срабатывает не статический блок инициализации
    
4. Внедрение зависимостей на основе конструктора
    
5. Внедрение зависимостей на основе `setter`-ов
    

Если вы используете комбинацию 2 подходов конфигурирования: на основе XML и на основе аннотаций, то следует знать, что внедрение зависимостей через аннотации выполняется перед внедрением зависимостей через XML. Таким образом, конфигурация XML переопределяет аннотации для свойств.

6. Отрабатывают методы стандартного набора `Aware` интерфейсов
    
7. `BeanPostProcessor#postProcessBeforeInitialization` обрабатывает bean-компонент
    
8. `InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization` вызывает методы обратного вызова, помеченные аннотацией `@PostConstruct`
    
9. `BeanFactory` вызывает метод `InitializingBean#afterPropertiesSet`
    
10. `BeanFactory` вызывает метод обратного вызова, зарегистрированный как `initMethod`
    
11. `BeanPostProcessor#postProcessAfterInitialization` обрабатывает bean-компонент
    
**![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXch-NaiE6Z_sAlWCLHE-6VHmG244d6-A1xdGr1rIkOeSuErmsTTRjj-axfNxxrj5MQFlg5jd-9MxqEPbL5GYxjbXL2jwj7S4FB2Y2lBnY7sejVOsbaQKogCJsV2FXlLEChFfMdBAyaFbvnGV-L-ChMDH_j3?key=XMeQwLql_JMyD2k54CogZg)**

Еще есть ContextListener. Он умеет слушать контекст и реагировать на его события: ContextStarter, ContextRefresh,...
В Boot можно вообще дополнительно настраивать контекст до его поднятия на этапе бутстрапа.
