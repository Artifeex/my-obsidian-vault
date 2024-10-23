Основные проблемы JDBC - это то, что трудозатратно писать маппинг данных, которые мы получаем из БД на java объекты. Еще одна проблема JDBC - это ручное написание простейших SQL запросов.

![[InputFiles/image 166.png|image 166.png]]

![[InputFiles/image 1 3.png|image 1 3.png]]

![[InputFiles/image 2 3.png|image 2 3.png]]

Создаем проект. Подключаем зависимости:

```Java
plugins {
    id 'java'
}

group = 'ru.sandr'
version = '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

dependencies {
    implementation 'org.hibernate:hibernate-core:5.5.6.Final' //hibernate core
    runtimeOnly 'org.postgresql:postgresql:42.2.23' //postgresql jdbc driver(runtime, т.к. нужен только в runtime, а не на этапе компиляции)

    compileOnly 'org.projectlombok:lombok:1.18.20' //аннотации ламбока, они нам не нужны на этапе компиляции
    annotationProcessor 'org.projectlombok:lombok:1.18.20' //процессор, который умеет обрабатывать аннотации

    testCompileOnly 'org.projectlombok:lombok:1.18.20' //те же зависимости на уровне тестов
    testAnnotationProcessor 'org.projectlombok:lombok:1.18.20'


    testImplementation platform('org.junit:junit-bom:5.7.2')
    testImplementation 'org.junit.jupiter:junit-jupiter'
}

test {
    useJUnitPlatform()
}
```

И коннектимся к БД, на линуксе через пользователя sandr с паролем 57933.

### 004 Конфигурация SessionFactory

Создали вот такую табличку, чтобы с ней поработать

![[InputFiles/image 3 3.png|image 3 3.png]]

Немного напомним, как мы поступали до этого. У нас был Connection Pool, в котором хранились соединения, так как создание нового соединения - это дорогостоящая операция. И дальше мы уже работали с Connection, у которого вызывали методы для получения preparedStatement и уже на нем вызывали. В Hibernate у нас есть соответствия этим сущностям.

Session соответствует connection, а для получения session мы используем SessionFactory. Т.е. SessionFactory соответствует Pool соединений.

![[InputFiles/image 4 3.png|image 4 3.png]]

Так же есть прямая аналогия между файликам с пропертис, который мы писали для подключения к БД. В hibernate так же есть такой файлик. Чтобы inteliji его нам добавила, заходим в настройки проекта и добавляем модуль Hibernate, а потом сам файлик

![[InputFiles/image 5 3.png|image 5 3.png]]

Главная задача конфигурационного файла - это задать параметры для SessionFactory, через которую мы будем получать Session

![[InputFiles/image 6 3.png|image 6 3.png]]

Внутри файлика указываем url, username, password и драйвер для работы с БД, который мы подключили ранее, используя gradle. А так же dialect, который позволяет hibernate сконфигурировать специфичные типы для конкретной БД.

![[InputFiles/image 7 3.png|image 7 3.png]]

`внутри метода configure() указываем путь до файла с конфигурациями, который мы добавили, но по умолчанию ищется название файла, как у нас, поэтому ничего не передаем`

![[InputFiles/image 8 3.png|image 8 3.png]]

В классе Configuration находится все, что нам нужно для создания SessionFactory. Он хранит в себе так же стратегии для маппинга java классов в таблицы БД, поддерживаемые типы

Получаем объект sessionFactory, используя configuration, который на основе файла с конфигурациями + своих полей создает нам SessionFactory.

![[InputFiles/image 9 3.png|image 9 3.png]]

![[InputFiles/image 10 3.png|image 10 3.png]]

И наконец получаем объект типа Session, который является оберткой на Connection, через который мы и работали с БД, когда использовали JDBC

![[InputFiles/image 11 3.png|image 11 3.png]]

### 005 Entity

Entity - это сущность(класс в java), который мы будем проецировать на таблицу в БД.

Создаем класс и мапим типы данных из Postgres на типы данных из java.

![[InputFiles/image 12 3.png|image 12 3.png]]

Чтобы класс соответстовал требованиям hibernate он должен быть POJO - plain old java object.

POJO:

- Все поля private и не final, т.к. hibernate будет постоянно менять значения полей, используя сеттеры.
- У всех полей есть геттеры и сеттеры
- Сам класс не может быть final, т.к. hibernate работает через прокси чаще всего(создает наследника нашего класса и работает с ним).
- В нем должен быть конструктор без параметров, так как hibernate использует reflection api, для создания объекта, и потом сеттеры для инициализации полей.
- Уже для нашего удобства добавляем конструктор со всеми параметрами, toString, equals, hashCode(т.к. часто сущности мы храним в каких-то коллекциях в нашем приложении).

И если посмотришь на полученный класс, то увидишь, что получишь очень много бойлерплейт кода.

```Java
public class User {

    private String username;
    private String firstname;
    private String lastname;
    private LocalDate birthDate;
    private Integer age;

    public User() {

    }

    public User(String username, String firstname, String lastname, LocalDate birthDate, Integer age) {
        this.username = username;
        this.firstname = firstname;
        this.lastname = lastname;
        this.birthDate = birthDate;
        this.age = age;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getFirstname() {
        return firstname;
    }

    public void setFirstname(String firstname) {
        this.firstname = firstname;
    }

    public String getLastname() {
        return lastname;
    }

    public void setLastname(String lastname) {
        this.lastname = lastname;
    }

    public LocalDate getBirthDate() {
        return birthDate;
    }

    public void setBirthDate(LocalDate birthDate) {
        this.birthDate = birthDate;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", firstname='" + firstname + '\'' +
                ", lastname='" + lastname + '\'' +
                ", birthDate=" + birthDate +
                ", age=" + age +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return Objects.equals(username, user.username) && Objects.equals(firstname, user.firstname) && Objects.equals(lastname, user.lastname) && Objects.equals(birthDate, user.birthDate) && Objects.equals(age, user.age);
    }

    @Override
    public int hashCode() {
        return Objects.hash(username, firstname, lastname, birthDate, age);
    }
}
```

Воспользуется ламбоком, чтобы избавиться от этого кода и получим такую конфету(еще решили использовать паттерн билдер, для красивого создания объектов)

```Java
@Data //создает геттеры, сеттеры, equals, hashCode, toString, еще какие-то конструкторы, но непонятно какие
@AllArgsConstructor 
@NoArgsConstructor
@Builder
public class User {

    private String username;
    private String firstname;
    private String lastname;
    private LocalDate birthDate;
    private Integer age;
}
```

Теперь, чтобы полученный класс стал Hibernate сущностью, нужно пометить его аннотацией Entity, а так же есть требование, чтобы обязательно был первичный ключ, для этого используется аннотацию Id. Причем, id должен реализовывать Serialiazable.

```Java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
public class User {

    @Id
    private String username;
    private String firstname;
    private String lastname;
    private LocalDate birthDate;
    private Integer age;
}
```

И теперь сохраним нашу сущность в БД:

```Java
public static void main(String[] args) throws SQLException {
        Configuration configuration = new Configuration();
        configuration.configure();

        try (SessionFactory sessionFactory = configuration.buildSessionFactory();
             Session session = sessionFactory.openSession();) {
            //в hibernate не autocommit мод, как в jdbc, поэтому мы явно должны создавать транзакции
            session.beginTransaction();

            User user = User.builder()
                    .username("artifeexs@gmail.com")
                    .firstname("Andrew")
                    .lastname("Smirnov")
                    .birthDate(LocalDate.of(2002, 3, 16))
                    .age(22)
                    .build();
            //сохраняем нашу сущность в БД
            session.save(user);
            //завершаем нашу сессию, чтобы выполнился код!
            session.getTransaction().commit();
        }

    }
```

Но, если мы щас сделаем так, то произойдет ошибка, так как Hibernate не знает, что нужно отслеживать сущность User(даже несмотря на то, что мы пометили класс аннотацией Entity). Чтобы Hibernate узнал о User есть 2 варианта:

1 варинат - добавить этот класс через объект configuration  
  
`configuration.addAnnotatedClass(User.class);`

2 вариант - это добавить этот класс через hibernate.cfg.xml:

![[InputFiles/image 13 3.png|image 13 3.png]]

Spring использует 1 вариант добавления сущностей.

Добавим еще 2 проперти, чтобы выводился sql код, сгенерированный hibernate:

![[InputFiles/image 14 3.png|image 14 3.png]]

Если запустим, то получим снова ошибку, как можем увидеть по сгенерированному коду, он пытается вставить в таблицу User(взял из названия java класса), а у нас таблица называется users.

![[InputFiles/image 15 3.png|image 15 3.png]]

Поэтому добаляем аннотацию Table, в которой указываем имя таблицы в постгрес и схему(могли не указывать, т.к. по умолчанию public).

![[InputFiles/image 16 3.png|image 16 3.png]]

Если снова попробуем запутить, то новая проблема, hibernate использовал название поля birthDate, а оно не соотвествует названию столбца в таблице birth_date.

![[InputFiles/image 17 3.png|image 17 3.png]]

Но мы хоти, чтобы camelCase в названиях полей переходил в snake_case в БД при маппинге. Тогда нам нужно изменить стратегию маппинга, которая хранится как раз в классе Configuration. Нас интересует physicalNamingStrategy(java → postgres)

![[InputFiles/image 18 3.png|image 18 3.png]]

Либо второй способ, это явно задать для каждого поля название из БД, используя аннотацию Column.

![[InputFiles/image 19 3.png|image 19 3.png]]

В Column можно много чего задавать

![[InputFiles/image 20 3.png|image 20 3.png]]

И вот после этого у нас уже все заработало!

Резюме:

1) Аннотация Entity, чтобы пометить класс для отслеживания Hibernate

2) Чтобы Hibernate начал следить за помеченным классом, нужно это явно сделать либо через xml, либо через configration.addAnnotatedClass

3) Аннотация Table - для задания названия таблицы, связанной с Entity

4) Аннотация Column для связи названия поля и названия столбца в таблице

5) Класс должен иметь конструктор без параметров + все геттеры и сеттеры

### 006 Класс Session

Session является оберткой над Connection и имеет много интересного функционала:

- update - обновить сущность в бд
- save - сохранить
- delete- удалить
- saveOrUpdate - сохранить или обновить в зависимости от того, есть переданная сущность или нет
- get - получить
- createQuery - если хотим сделать какой-то свой SQL запрос

Раньше нам приходилось писать свой SQL запрос, потом с помощью connection.prepareStatement(SQL_QUERY_STRING) получать preparedStatement. Потом у этого preparedStatement вставлять значения в поля запроса(где были ?), делая как раз маппинг из java объекта в тип для БД. Но теперь все это за нас делает hibernate. Как он это делает? Использует ReflectionApi.

Вот, что примерно происходит под капотом:

```Java
String sql = """
                insert
                into
                %s
                (%s)
                values
                (%s)
                """;
        
        String tableName = Optional.ofNullable(user.getClass().getAnnotation(Table.class))
                .map(tableAnnotation -> tableAnnotation.schema() + "." + tableAnnotation.name())
                .orElse(user.getClass().getName()); //если не было задана аннотация Table, то используем имя поля

        Field[] declaredFields = user.getClass().getDeclaredFields();

        String columnNames = Arrays.stream(declaredFields)
                .map(field -> Optional.ofNullable(field.getAnnotation(Column.class))
                        .map(Column::name)
                        .orElse(field.getName()))
                        .collect(joining(", "));

        String columnValues = Arrays.stream(declaredFields)
                .map(field -> "?")
                .collect(joining(", "));

        System.out.println(sql.formatted(tableName, columnNames, columnValues));
```

И остается только вставить значения в preparedStatement, который мы создадим на основе полученного SQL, здесь приведена примерная схема, как бы мы это делали:

![[InputFiles/image 21 3.png|image 21 3.png]]

### 007 Type converters

timestamp в postgres - это дата + время.

Как Hibernate конвертирует наши java типы, если они не подходят под SQL типы, например, у нас есть LocalDateTime поле, но JDBC принимает Timestamp, в который нужно сначала конвертнуть наш LocalDateTime.

Внутри класса Configuration есть поле basicTypes, в котором перечисляются все поддерживаемые типы и он может преобразовывать один тип в другой.

![[InputFiles/image 22 3.png|image 22 3.png]]

BasicType - реализует интерфейс Type. И все поддерживаемые типы в Hibernate реализуют этот интерфейс. Например, есть класс LocalDateType и в Hibernate есть LocalDateJavaDescriptor класс, в котором два метода wrap и unwrap. wrap - врапит полученное из SQL тип(то, что поддерживает JDBC) в наш Entity тип поля. unwrap наоброт.

И есть еще DateTypeDescriptor - который уже устаналивает значения в preparedStatement, а также умеет из ResultSet получить значения.

Но если заинтересовало, то лучше еще раз более вдумчиво посмотреть.

Так же Hibernate автоматически поддерживает enum.

Создадим свой enum и добавим его в java класс:

![[InputFiles/image 23 3.png|image 23 3.png]]

![[InputFiles/image 24 3.png|image 24 3.png]]

Добавим столбец role в БД(здесь можно либо VARCHAR либо INT, поскольку в enum есть методы ordinal, вернет число и name, вернет название enum).

Но лучше использовать в БД строкове представление, а не INT, для этого нужно сделать тип в БД varchar и добавить аннотацию нам Enum:

![[InputFiles/image 25 3.png|image 25 3.png]]

### 008 Custom attribute converter

На прошлом уроке мы узнали, как происходит конвертация java типов в sql. Но, что, если мы хотим написать свой тип, который Hibernate должен уметь конвертировать?

Для этого мы могли бы реализовать у нашего класса интерфейс Type, но он огромный и сложный. Есть более простой вариант - это реализовать конвертер.

У нас было ненужное поле age, поскольку мы можем узнать возраст по дате рождения.

Создадим новый класс:

![[InputFiles/image 26 3.png|image 26 3.png]]

Добавим его в нашего user, удалив age поле и birth_date

![[InputFiles/image 27 3.png|image 27 3.png]]

Создаем конвертер, который будет преобразовывать созданный нами класс:

![[InputFiles/image 28 3.png|image 28 3.png]]

И еще мы должны подсказать Hibernate, чтобы он использовал написанный нами конвертер для поля birthday.

![[InputFiles/image 29 3.png|image 29 3.png]]

Но, мы можем сделать так, чтобы не приходилось добавлять аннотацию Convert, а чтобы всегда, когда HIbernate видел поле Birthday у своих сущностей, то он использовал конвертер. Для этого через configuration добавляем конвертер, а также ставим в true флаг autoApply(из названия понятно, что автоматически применять конвертер).

![[InputFiles/image 30 3.png|image 30 3.png]]

Вместо передачи autoApply = true, можем навесить аннотацию над классом конвертер

![[InputFiles/image 31 3.png|image 31 3.png]]

### 009 Custom user type

На прошлом занятии мы узнали, как использовать конвертеры для наших типов данных.

Но, что, если в SQL нет подходящего типа для нашего типа данных. В прошлый раз для нашего класса Birthday было соответсвие Date. Но, такого Date может и не быть, тогда мы будем сохранять массив байт, а тут уже конвертером не обойдешься. Или наоброт, может не быть аналога из БД в наш java.sql тип.

![[InputFiles/image 32 3.png|image 32 3.png]]

![[InputFiles/image 33 3.png|image 33 3.png]]

И в java.sql нет какого-то класса, который бы подошел для типа JSONB

Создадим свой тип, который будет соотвествовать. Как говорили ранее, есть вариант реализовать интерфейс Type, но можно реализовать упрощенный Type - это интерфейс UserType, который реализует часть методом из Type, за счет чего нам меньше нужно реализовать

![[InputFiles/image 34 3.png|image 34 3.png]]

![[InputFiles/image 35 3.png|image 35 3.png]]

Но помимо этих методов есть множество других методов, которые тоже нужно реализовать. Поэтому есть библиотеки, которые реализуют различные типы из БД. ОДна из таких - hiberate-types

![[InputFiles/image 36 3.png|image 36 3.png]]

В Hibernate-types есть класс JsonBinaryType и его мы и будем использовать.

Мы должны зарегестрировать новый тип, для этого используем Configuration.  
  
`configuration.registerTypeOverride(new JsonBinaryType());`

А также пометить поле в нашем классе User, которое будет соответствовать JsonB из postgres. В type передается полный путь до класса.

![[InputFiles/image 37 3.png|image 37 3.png]]

Теперь можем добавлять json:

![[InputFiles/image 38 3.png|image 38 3.png]]

Можно передавать не полный путь, а т.к. в классе JsonBinaryType реализован метод name(), который достался от интерфейса Type, то в этом методе как раз jsonb и поэтому hibernate воспользуется этим методом у всех своих Types и найдет как раз jsonb и свяжет с этим полем, котором мы пометили.

![[InputFiles/image 39 3.png|image 39 3.png]]

И все работает:

![[InputFiles/image 40 3.png|image 40 3.png]]

### 010 Методы updatedeleteget

Рассмотрим другие методы объекта session.

- session.update(user) - обновляет поля для переданного объекта, если такого объекта не будет в БД, то выбросится исключение.

Так же очень важно заметить, что в Hibernate отложенное выполнение запросов. Т.е. даже когда мы вызвали update никакой запрос в БД не пошел. Запрос в БД выполнится только тогда, когда мы закоммитим транзакцию(session.getTransaction().commit()). Это нужно для оптимизации, чтобы одновременно можно было посылать batch запросов.

- saveOrUpdate(user) - либо обновить, либо создать нового, если не оказалось переданного юзера. О том, есть или нет сущности hibernate понимает, когда делает доп запрос select, где в where ищет по первичному ключу, который есть у любой сущности, когда мы помечаем поле аннотацией Id
- session.delete(user) - если сущности не будет, то ничего не произойдет(т.е. без исключений, как в случае update())
- User user = session.get(User.class, первичный ключ); User.class мы передаем потому что get метод должен знать, какую сущность ему нужно создать + первичный ключ есть в куче других таблиц, как Hibernate узнает по какой таблице мы ищем ? Это так же благодаря тому, что мы передаем во внутрь User.class.

Под капотом в случае get метода происходит примерно следующее:

![[InputFiles/image 41 3.png|image 41 3.png]]

### 011 EntityPersister

Остановимся поподробнее на методе session.get().

EntityPersister маппит sql запросы с нашей сущностью. Внутри factory есть класс Metamodel с метаинформацией в котором есть ассоциативный массив, у которого ключом является полный путь до нашей сущности, а значением - EntityPersister нашей сущности, который занимается обновлением, добавлением и т.д. в общем всей работой с нашей сущностью.

В Metamodel(session.getFactory().getMetaModel()) так же хранит информацию о зарегистрированных типах, в том числе тот тип, который мы добавили тогда jsonb.

Внутри метода get мы получаем entityPersister переданной сущности, а у entityPersister есть различные методы, например, load, который как раз получает данные из БД.

И получается, что у каждой сущности будет свой entityPersister.

![[InputFiles/image 42 3.png|image 42 3.png]]

MetaModel строится на основе сущностей, которые мы имеем.

И еще hibernate работает на листенерах, т.е. есть листенер, который ждет, когда выполнится запрос и когда выполнится, то выполняется(ну аналогично с обработкой мышкой и т.д. такие листенеры, обработчики событий другими словами).

### 012 First Level Cache

EntityPersister отвечает за операции над своей сущностью и преобразует объектую модель в реляционную и наоборот. И класс нашей сущности является ключом в MetaModel, по которому мы получаем EntityPersister.

Попробуем получить одного и того же юзера:

![[InputFiles/image 43 3.png|image 43 3.png]]

При первом get запрос выполнится, а при втором уже нет, потому что используется first level cache. Он есть всегда по умолчанию и его никогда не отключить.

Внутри сессии есть persistanceContext, который и называют firstLevelCache

![[InputFiles/image 44 3.png|image 44 3.png]]

В persistanceContext есть ссылка на сессию, которой он принадлежит из этого делаем вывод, что у каждой сессии есть свой persistance context. Если у нас несколько сессий, то у каждой будет свой персистенс контекст.

Внутри персистенс контекста есть ассоциативные массивы, которые по ключу получают нашу сущность.

![[InputFiles/image 45 3.png|image 45 3.png]]

![[InputFiles/image 46 3.png|image 46 3.png]]

И получается, что если в persistanceContext уже есть сущность в ассоциативном массиве по ключу, то второй запрос делать не нужно, а нужно взять значение из этого ассоциативного массива.

entitiesByKey - в persistanceContext!!!

![[InputFiles/image 47 3.png|image 47 3.png]]

В персистенс контекст кладутся значения только тогда, когда был сделан запрос. Т.е. персистенс контекст отображает реальное значение в БД в текущий момент.

А если захотим удалить сущность из персистенс контекста?

Есть 3 способа

```Java
session.evic(user1); - так мы из кэша(из персистенс контекста) удалим нашу сущность user1, которая туда должна была попасть ранее, вызовом, например, get
session.clear() - чистит весь персистенс контекст(кэш)
session.close() - вызывается автоматически у сессии, когда мы выходим из блока try with resource и тогда так же очищается весь кэш.
```

Еще одно важное открытие - это то, что все сущности, которые лежат внутри persistanceContext нашей сессии влияют на будущие запросы. На скрине ниже, мы получили пользователя, он загрузился в персистенс контекст и теперь, мы просто у этого пользователя меняем значение поля и это изменение отобразится в БД! Мы не делаем никакие update, но это изменение отобразится! Т.е. когда мы сделаем commit hibernate вызовет метод update и обновит значение в БД!!! И это очень очень очень очень важно). В таком случае, сессия становится грязной(ну вот так называют сессию, в которой происходит изменение объекта, находящегося в персистенс контексте).

![[InputFiles/image 48 3.png|image 48 3.png]]

session.flush() - когда мы все наши изменения в объектах сливаем в БД(делаем соотвествующие SQL запросы). Т.е. мы синхронизируем состояние нашего first level cache с БД. Это автоматически происходит при session.close(), но можем вызвать самостоятельно.

Т.е. получаем такую схему, что внутри SessionFactory есть множество сессий(некоторый пул) и у каждой сессии есть свой кэш. И важно, что сущности могут находится в разных персистенс конекстах разных сессий и иметь разные состояния(но об этом позже).

![[InputFiles/image 49 3.png|image 49 3.png]]

### 013 Entity lifecycle. Теория

![[InputFiles/image 50 3.png|image 50 3.png]]

Состояние Transient - сущность ни с какой сессией не ассоциирована. Т.е. мы просто создали нашу сущность через какой-нибудь конструктор. Чтобы сохранить сущность в персистенс конексте всегда нужно, чтобы вызвался какой-то метод, который бы связал данную сущность с персистенс конекстом. В случае новой сущности мы должны вызвать внутри сессии метод save или saveOrUpdate(). Тогда из transient состояние сущности переходит в состояние Persistent - это значит, что сущность связана с persistent контекстом нашей сессии. Именно той сессии, в которой вызвали метод save. Т.е. если есть параллельно еще какая-то сессия и в ней мы не вызывали метод save для нашей сущности, то для другой сессии сущность будет находиться в состоянии transient, а для нашей сессии, в которой мы вызвали save будет находиться в состоянии persistent.

Еще в персистент состояние сущности могут перейти, если мы их получаем сразу же из БД, вызовет get на сессии. Так же неявно сущности попадают в персистенс состояние, например, когда вызываем метод update, то, чтобы понять, есть ли такая сущность или нет в БД hibernate под капотом вызовет метод get, чтобы попытаться получить сущность.

Далее мы могли вызвать session.delete(entity) и тогда наша сущность удаляется из персистенс контекста нашей сессии и из БД. delete так же мгновенно не исполняется, так как отложенное выполнение запросов, но если все-таки хотим, чтобы сразу выполнилось, то вызываем session.flush().

detached - состояние, в котором можно попасть только вызовом методов evict, clear, close, которые мы в прошлый раз рассматривали. Т.е. мы удаляем сущность из нашего пересистенс контекста.

detached отличается от transient тем, что если сущность имеет состояние detached, значит, она когда-то была в персистенс контексте, но удалилась из него. А transient никогда не была в нем.

Так же из detached состояния мы можем вернуться обратно в persistent если вызовем saveOrUpdate, update, merge(но в любом случае под капотом вызовется get, из-за чего сущность и попадает обратно в persistent context).

### 014 Entity lifecycle. Практика

![[InputFiles/image 51 3.png|image 51 3.png]]

Создадим утилитный класс(т.е. будет статические методы его вызывать) для конфигурирования объекта Configure и получения SessionFactory:

```Java
@UtilityClass
public class HibernateUtil {

    public static SessionFactory buildSessionFactory() {
        Configuration configuration = new Configuration();
        //configuration.addAnnotatedClass(User.class);
        configuration.setPhysicalNamingStrategy(new CamelCaseToUnderscoresNamingStrategy());
        configuration.addAttributeConverter(new BirthdayConverter());
        configuration.registerTypeOverride(new JsonBinaryType());
        configuration.configure();
        return configuration.buildSessionFactory();
    }
}
```

Тут показано в каком состоянии будет entity по отношению к двум сессииям в разные моменты времени.

```Java
public static void main(String[] args) throws SQLException {
        User user = User.builder()
                .username("artifeexs1@gmail.com")
                .firstname("Andrew")
                .build();
        //transient, transient (session1, session2)
        try (SessionFactory sessionFactory = HibernateUtil.buildSessionFactory()) {
            try(Session session1 = sessionFactory.openSession()) {
                session1.beginTransaction();

                session1.saveOrUpdate(user); //persistent, transient

                session1.getTransaction().commit();
                //detached, transient
            }

            try(Session session2 = sessionFactory.openSession()) {
                session2.beginTransaction();
                //session1 уже нет, но если бы была, то detached. У сессии 2 же entity состояние = transient


                session2.delete(user);
                //persistent состояние у entity в session2
                session2.getTransaction().commit();//entity будет иметь removed состояние для session2

            }

        }
    }
```

Метод session.refresh(user) - берет данные из БД для переданного пользователя(т.е. выполняет get) и дальше для нашего JAVA объекта обновляет значения всех полей теми, которые находятся в БД. На скрине ниже 3 первые строчки показывают то, что делает метод session2.refresh(user) под капотом. Если бы у нас был как-нибудь изменен java объект, при этом он не был в persistent состоянии, а потом мы для этого java объекта вызвали метод refresh, то все изменения, которые мы сделали над java объектом откатятся и он будет иметь то же состояние, что и в БД.

![[InputFiles/image 52 3.png|image 52 3.png]]

Метод session.merge(user) - очень похож на метод refresh, но данные в java объекте главнее. Т.е. он возьмет данные по сущности из БД, получит объект и затем у этого объекта с помощью set установит значения из объекта user, который мы передали в метод merge. Т.е. произойдет следующее(3 строчки снова показывают, что происходит под капотом):

![[InputFiles/image 53 3.png|image 53 3.png]]

В mergedUser будут значения из БД + измененные поля, если в user были изменены значения полей. Но, теперь наша сессия стала dirty, так как мы получили пользователя из БД и потом с помощью сеттера задали значения, поэтому после комита данные обновятся и в БД. - это в случае merge. В случае же refresh данные в БД не обновятся.

![[InputFiles/image 54 3.png|image 54 3.png]]

Резюмируя merge и refresh - refresh обновляет java объект данными из БД. merge же обновляет значения в БД по состоянию нашего объекта(происходит за счет того, что новый объект, который вернул нам merge попал в persistent состояние, а после этого вызвались сеттеры, которые изменили значения полей и тем самым сессия стала dirty).

### 015 Java Persistence API (JPA)

![[InputFiles/image 55 3.png|image 55 3.png]]

JPA - важно, что это именно набор ИНТЕРФЕЙСОВ И АННОТАЦИЙ, т.е. нет никаких реализаций, которые мы можем использовать. Поэтому Hibernate - это набор классов и аннотаций свой, которые направлены на то, чтобы реализовать JPA.

Это все равно, что есть интерфейс List(JPA), а Hibernate реализация List(ArrayList).

Как это отражается в коде?

Например, зайдем в интерфейс SessionFactory, он лежит в пакете org.hibernate, но мы видим, что он экстендит интерфейс EntityManagerFactory

![[InputFiles/image 56 3.png|image 56 3.png]]

И мы видим, что интерфейс EntityManagerFactory из пакета javax.persistence.

![[InputFiles/image 57 3.png|image 57 3.png]]

И оказывается, что мы часто видели этот пакет, например, аннотации для сущностей так же из этого пакета!

![[InputFiles/image 58 3.png|image 58 3.png]]

Этот пакет достался нам транзитивно вместе с зависимостью для hibernate. И это как раз JPA спецификация - набор интерфейсов и аннотаций, которые можно использовать и hibernate как раз их и использует.

И например Session интерфейс экстендится от EntityManager и реализация интерфейса SessionImp должна реализовать все методы и поэтому в Session есть как методы из hibernate, так и методы из JPA, которые обязаны реализовать из-за того, что extends EntityManager. И тогда на сессии мы можем вызывать как hibernate методы, так и jpa методы. Разработчики hibernate просто создали доп методы с названиями, которые им показались более хорошими, чем те, что предоставляет JPA. Поэтому зачастую hibernate метод под капотом вызывает реализованный JPA метод.

Например, find() из JPA делает то же самое, что и get из hibernate. save = persist. delete=remove.

### 2.001 Logging. Теория

Когда мы запускали код, то замечали, что выводится множество сообщений - это система логирования в hibernate:

![[InputFiles/image 59 3.png|image 59 3.png]]

Если мы не используем никакую систему логирования об ошибках, например, то если что-то пойдет не так в приложении, то мы об этом даже не узнаем, узнают недовольные пользователи, которые придут за нами :) Поэтому без логинования наше приложение - это black box.

![[InputFiles/image 60 3.png|image 60 3.png]]

И чтобы перейти к white box, нужно использовать метрики и логирование. Логирование - это какая-то информация о происходящих событиях(какие-то сообщения), а в случае метрик - это статистика о нашем приложении(какая нагрузка на приложение, средняя длительность ответа для пользователя, обращение к БД и т.д.)

![[InputFiles/image 61 3.png|image 61 3.png]]

Самое простейшее логирование - это println(). Но это работает только тогда, пока мы разрабатываем приложение, потому что у нас есть консоль, в которую мы выводим. Поэтому если приложеине запущено удаленно, то доступа к консоли у нас уже нет, поэтому используются файлы, в которым можно складывать сообщения и потом, когда понадобится человек сможет почитать в этом файле логи. Но это тоже неудобно, т.к. в реальных прилоежниях может проихсодить тысячи событий в секунду и тогда будут огромные файлы, которые будет сложно читать и находить информацию. Поэтому можно сохранять в БД и с помощью запросов получать какие-то данные. Но в реальных приложениях пошли еще дальше и посылают логи какому-то удаленному приложению, которое уже их собирает и умеет всячески анализировать и выдавать по запросу.

Для логов самые популярные - ELK.

Для метрик - Prometheus, Grafana.

Есть много систем для логирования и slf4j api - это как JPA, т.е. набор интерфейсов, которые можно использовать, а вот, например, log4j, это реализация, как hibernate.

Другими словами, мы будем пользоваться api, которое нам предоставляет slf4j, а для реализации этого api подставим одну из системы логов на слайде. bingind на слайде - это значит, что есть адаптер, через который slf4j будет общаться, а для logback, например, не нужен никакой адаптер и все вызовы slf4j api будут идти сразу на logback.

![[InputFiles/image 62 3.png|image 62 3.png]]

Для того, чтобы разделить сообщения и их было проще группировать и фильтровать придумали лвлы логов. Чем выше уровень, тем ошибка жесте(самый низкий уровень - trace). При использовании же, мы можем указывать минимальный уровень ошибок, который хотим видеть, например, можем указать уровень error, тогда будем видеть ошибки error и fatal, а другие не будем(в нашем простом случае, они просто не будут отображаться на консоли).

![[InputFiles/image 63 3.png|image 63 3.png]]

Ранее, когда мы подключали hibernate мы еще подключили slf4j api депенденси, потому что hibernate требует этого(он как раз в консоль и выводит различные логи, как мы могли заметить).

Но, чтобы нам работать с какой-то из реализаций slf4j api, нам нужно подключить доп зависимости. В случае log4j нам нужно binding(adapter, чтобы slf4j знала, как общаться с log4j).

![[InputFiles/image 64 3.png|image 64 3.png]]

![[InputFiles/image 65 3.png|image 65 3.png]]

Множество binding логеров, если просто перейдем в slf4j:

![[InputFiles/image 66 3.png|image 66 3.png]]

### 2.002 Logging. Файл log4j.xml

Каждый логер конфигурируется файликом, который лежит в resources и имеет формат либо properties либо xml. В случае log4j:

[https://github.com/dmdev2020/hibernate-starter/tree/lesson-16](https://github.com/dmdev2020/hibernate-starter/tree/lesson-16)

appender берет лог сообщения и отправляет туда, куда укажем. В данном случае ConsoleAppender отправляет лог на консоль и мы будем видеть лог в стандартном потоке вывода. Т.е. как будто написали System.out.println. А формат сообщения указывается в layout. В PatternLayout есть расшифровка того, что за символы используются. Вместо этих символов будут подставляться реальные значения.

```Java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<log4j:configuration>
		
    <appender name="console" class="org.apache.log4j.ConsoleAppender">
        <param name="target" value="System.out"/>

        <layout class="org.apache.log4j.PatternLayout">
            <param name="conversionPattern" value="[%d{HH:mm:ss,SSS}] %p [%c: %L] %m%n"/>
        </layout>
    </appender>

    <root>
        <level value="trace"/>
        <appender-ref ref="console"/>
    </root>

</log4j:configuration>
```

Вот класс ConsoleAppender, видим, что тут указывается куда выводить в target.

![[InputFiles/image 67 3.png|image 67 3.png]]

Если подняться вверх по иерархии, то дойдем до главного интерфейса Appender, где есть различные методы, например, по добавлению фильтров или добавлению логов, которые представляют из себя класс LoggingEvent.

В LoggingEvent есть лвл, сообщение, имя треда, время сообщения и т.д.

< root> - это сам логер, где мы указываем log level, с которого мы начинаем логировать наши сообщения (<level value=”info”/>)

< appender-ref - указываем название нашего appender, который мы определили ранее). Аппендер, как и логеров может быть много. Т.е. root или же сам логер решает, на какие аппендеры отправлять логи.

![[InputFiles/image 68 3.png|image 68 3.png]]

Добавляем логер. Чаще всего на каждый класс свой логер, так как в логе сообщения часто указывают класс из которого исходил лог.

![[InputFiles/image 69 3.png|image 69 3.png]]

![[InputFiles/image 70 3.png|image 70 3.png]]

Теперь можем выполнять само логирование. У log множество методов в зависимости от уровня лога, например, info - уровень info, есть и другие методы, соотвествующие другим уровням. Внутрь мы передаем сообщение, которое хотим донести, при этом важно не использовать конкатенацию, а использовать curly скобочки, и потом после сообщения передаем параметры и сколько curly скобочек в сообщении, столько и параметров должно быть передано. Вместо скобочек будут вставляться значения из параметров в том порядке, в котором мы их передали. Важно не использовать конкатенацию, так как это тратит ресурсы, а логгер может даже не отслеживать такое сообщение, а ресурсы уже будут потрачены на конкатенацию строк.

![[InputFiles/image 71 3.png|image 71 3.png]]

Мне показалось необычным, что можно создать сессию и потом отдельно в try with resource передать объект, чтобы не потерять объект после того, как выйдем из блока try with resource.

![[InputFiles/image 72 3.png|image 72 3.png]]

В случае ошибок примерно такое поведение. Логируем ошибку и пробрасываем дальше. exception передаем, чтобы вывести stack trace

![[InputFiles/image 73 3.png|image 73 3.png]]

И если запустим, то увидим log:

![[InputFiles/image 74 3.png|image 74 3.png]]

### 2. 003 Logging. File appender

До этого мы выводили на консоль логи, что не очень интересно, так как и так могли выводить на консоль. Вся мощь логеров в том, что можем в разные места отправлять логи, даже по TCP или в БД.

Давайте попробуем сохранить лог к файл. Для этого нужно добавить новый appender

```Java
<appender name="file" class="org.apache.log4j.RollingFileAppender">
        <param name="file" value="hibernate-starter.log"/> //указываем путь и название файла(если не указали путь, то будет в рутовой лежать)

        <param name="append" value="true"/> //Каждое новое сообщение будет добавляться, а не перетираться, после рестарта нашего приложения
        <param name="maxFileSize" value="20MB"/>
        <param name="maxBackupIndex" value="10"/> //когда файл дойдет до максимального размера, то будут создаваться новые файлы и их макс. кол-во в этом параметре

        <layout class="org.apache.log4j.PatternLayout"> //снова паттерн логов, который у нас будет
            <param name="conversionPattern" value="[%d{yyyy-MM-dd HH:mm:ss,SSS}] %p [%c: %L] %m%n"/>
        </layout>

        <filter class="org.apache.log4j.varia.LevelRangeFilter"> //дополнительная фильтрация сообщений, не взирая на то, что мы пишем в логерах(root)
            <param name="LevelMin" value="ALL"/>
        </filter>
    </appender>
```
И чтобы сообщения выводились в file, я должен еще в рутовой логере указать, что используется такой аппендер. Тогда логи будут на консоли + file.

![[InputFiles/image 75 3.png|image 75 3.png]]

После запуска создалось множество файлов.

![[InputFiles/image 76 3.png|image 76 3.png]]

Так же мы можем создавать множество логеров. root логер может быть только один, для создания доп логеров используется тег logger. Название логгера - это имя пакета, в котором он будет работать и во всех нижележащих пакетах. Т.е. в нашем случае, в логгер будут приходиться сообщения из всех классов в пакете com.dmdev и нижележащих пакетах.

![[InputFiles/image 77 3.png|image 77 3.png]]

При запуске сообщения дублируются, так как оба логера отправляют сообщения аппендеру console. Чтобы этого не было, можем указать additivity=false.

![[InputFiles/image 78 3.png|image 78 3.png]]

Так же у логеров тоже есть иерархия. root - главный логер и все логгеры наследуются от него.

Так же логгеры наследуются по иерархии имен пакетов, которые они слушают. com.dmvdev.entity наследуется от com.dmdev и если они будут посылать сообщения одному аппендеру, то снова будет дублирование, поэтому опять нужно ставить additivity =false. Но на практике редко используют много логеров, чаще всего парочку.

![[InputFiles/image 79 3.png|image 79 3.png]]

И в lambok есть аннотация, которая позволяет не дублировать поле Logger log. Мы можем удалить это строку и обращаться к объекту log

![[InputFiles/image 80 3.png|image 80 3.png]]

Удалили поле, но при этом можем использовать:

![[InputFiles/image 81 3.png|image 81 3.png]]

### 3. 001 Embedded components

Часто на уровне джава мы работаем со сложными объектами, которые внутри себя содержат другие поля и мы можем захотеть в сущности объединить какие-то поля в одно поле какого-то объекта. Но мы не хотим создавать какой-то новый тип в БД.

Давайте попробуем объединить эти 3 выделенных поля в один java класс:

![[InputFiles/image 82 3.png|image 82 3.png]]

Создадим класс PersonalInfo и объединим в нем поля:

![[InputFiles/image 83 3.png|image 83 3.png]]

И чтобы показать, что это встраиваемый объект, мы используем аннотацию Embeddable. И внутри встраиваемого компонента мы можем использовать все аннотации Hibernate для названия полей и всего такого.

![[InputFiles/image 84 3.png|image 84 3.png]]

И теперь добавляем в User класс созданное поле:

![[InputFiles/image 85 3.png|image 85 3.png]]

При этом названия полей в embedded объекте важны и мы либо используем такое же название, как в БД, либо используем аннотацию Column так же, как делали это для обычной Entity, либо 3-ий способ: И т.к. это аннотация Repeatable, то мы можем повторить ее несколько раз для каждого поля нашего Embedded объекта.

![[InputFiles/image 86 3.png|image 86 3.png]]

### 3. 002 Primary keys

В реальных приложениях чаще всего используются автосгенерированные первичные ключи.

Изменим табличку Users и пусть в нем будут автосгенерированные ключи

![[InputFiles/image 87 3.png|image 87 3.png]]

Т.к. теперь мы должны сказать Hibernate, чтобы он не клал в Id никакое значение при SQL запросах на сохранение, поскольку мы сказали БД автоматически генерировать это поле с помощью serial, то указываем на это с помощью аннотации GeneratedValue

![[InputFiles/image 88 3.png|image 88 3.png]]

4 возможных GenerationType:

![[InputFiles/image 89 3.png|image 89 3.png]]

- AUTO - в зависимости от выбранной СУБД, диалекта и других штук будет использовать одну из оставшихся 3 типов
- IDENTITY - будем использовать чаще всего, используется, когда у primary key в БД мы используем тип с окончанием Serial и тогда сама БД занимается инкрементом sequnce самостоятельно при вставке.
- SEQUENCE.
    
    - Когда мы создали id и сказали BigSerial, то автоматически создался сиквенс, который как раз и автоинкрементируется и выдает новые id. Мы можем и самостоятельно создавать сиквенсы.
    
    ![[InputFiles/image 90 3.png|image 90 3.png]]
    
    Тогда, если мы используем созданный нами сиквенс, то дополнительно нужно использовать аннотацию SequnceGenerator, которая свяжет наше поле с сиквенсом.
    
    ![[InputFiles/image 91 3.png|image 91 3.png]]
    
    В таком случае немного меняется SQL код и мы видим, что сначала вызывался nextval, чтобы получить значение сиквенса и инкрементнуть его и потом при инсерте вставить значение id.
    
    ![[InputFiles/image 92 3.png|image 92 3.png]]
    
- Table - используется реже всего, если БД не поддерживает ни сиквенсы, не автогенерированные keys.
    
    - В такое случае создается таблица, в которой primary key - это название таблицы, а также в таблице хранится счетчик для каждой таблицы
    
    ![[InputFiles/image 93 3.png|image 93 3.png]]
    
    Тогда мы используем аннотацию TableGenerator
    
    ![[InputFiles/image 94 3.png|image 94 3.png]]
    

Небольшое дополнение по работе GenerationType.SEQUENCE vs GenerationType.IDENTITY. Чтобы сущность перешла из transient состояния в persistent ей должен быть установлен id. В случае сиквенс сначала выполняется запрос к сиквенсу и получаем id, устанавливаем его, но остальной запрос не выполнится, пока не пройзоидет flush сессии. В случае же IDENTITY мы сразу же выполняем запрос на INSERT, т.к. другими способами id не узнать, из-за того, что ключ автогенерируется БД. (хз, важно или нет).

Вывод: лучше всего использовать Identity

### 3. 003 EmbeddedId

В легаси базах данных могут быть составные первичные ключи, что делать в таком случае ?

Изменим БД и сделаем составной первичный ключ:

![[InputFiles/image 95 3.png|image 95 3.png]]

Мы хотим, чтобы объект типа PersonalInfo был первичным ключом(внутри он как раз содержит эти поля, которые мы сделали составным первичным ключом). Используем аннотацию EmbeddedId в такое случае и нужно еще поправить класс PersonalInfo

![[InputFiles/image 96 3.png|image 96 3.png]]

Класс для первичного ключа должен быть Serialiazable.(напоминание первичный ключ - это UNIQUE + NOT NULL)

![[InputFiles/image 97 3.png|image 97 3.png]]

После этого все работает, просто в качестве ключа используем составной объект. Но так делать не стоит, лучше использовать IDENTITY.

### 3. 004 Other basic annotations

Аннотация Transient над полем говорить Hibernate не сохранять это поле в БД(ну и не получать его тоже). Но лучше не использовать, т.к. в сущности лучше хранить только те поля, которые есть в БД.

Соответствия между типов времени в java и в postgres:

LocalDate - Date

LocalTime - Time

LocalDateTime - TimeStamp

А раньше, когда не было этих новых типов, а был только класс Date, то нужно было указывать аннотацию Temporal(TemporalType.TIMESTAMP) и другие, чтобы Hibernate понимал как именно мапить класс из БД. Т.е. внутри Temporal мы указывали тот тип, который имела колонка в БД.

Аннотация ColumnTransformer нужна для того, чтобы переданный SQL код вызывался при чтении или записи в этот столбец. На примере ниже у нас есть поле credit_card_num и мы хотим при чтении вызывать встроенную функцию decrypt, чтобы в сущности была расшифрованное значение. Аналогично для write.

![[InputFiles/image 98 3.png|image 98 3.png]]

Аннотация Formula - тоже самое, что и ColumnTransformer, но отличие в том, что мы внутри аннотации можем указать только тот код, который будет вызываться при чтении колонки из БД. Но если уж хотим такой функционал, то лучше использовать ColumnTransformer.

![[InputFiles/image 99 3.png|image 99 3.png]]

### Еще раз можно посмотреть видео про Persister и метод byId, который получает какой-то объект и у него уже вызывается метод load().

### 4. 001 ManyToOne

Работаем с такими таблицами:

![[InputFiles/image 100 3.png|image 100 3.png]]

Класс для таблицы Company:

```Java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
//@Table(name = "company") //Но эта аннотация необязательна, так как постгрес не чувствителен к регистру таблиц
public class Company {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Integer id;

    private String name;
}
```

Класс для таблицы Users:

```Java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name = "users", schema = "public")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true) //unique - метаинформация и необязательна
    private String username;

    @Embedded //но это необязательная аннотация, а название поля вообще неважно для PersonalInfo
    private PersonalInfo personalInfo;

    @Type(type = "jsonb")
    private String info;

    @Enumerated(EnumType.STRING)
    private Role role;

    @ManyToOne //много юзеров к одной компании, которая как раз и идет после аннотации One
    @JoinColumn(name = "company_id") //name-название колонки(foreignKey) в таблице user
    private Company company;
}
```

```Java
public class HibernateRunner {
    
    public static void main(String[] args) throws SQLException {
        Company company = Company.builder()
                .name("Google")
                .build();
        User user = User.builder()
                .username("artifeexs1@gmail.com")
                .personalInfo(PersonalInfo.builder()
                        .firstname("Andrew")
                        .lastname("Smirnov")
                        .birthday(new Birthday(LocalDate.of(2002, 3, 16)))
                        .build())
                .company(company)
                .build();
        try (SessionFactory sessionFactory = HibernateUtil.buildSessionFactory()) {
            try (Session session1 = sessionFactory.openSession()) {
                session1.beginTransaction();
                //т.к. для сохрания user нам нужен id компании, поэтому сначала должны сохранить компанию
                session1.save(company); 
                session1.save(user);

                session1.getTransaction().commit();

            }
        }
    }
}
```

Теперь при вызове save() сразу делается insert, т.к. у Id GeneratedType=IDENTITY, поэтому должны забрать из БД при инсерте.

### 4. 002 Fetch types

С кодом, который мы написали ранее, если мы вызовет session.get(User.class, id), то в User заполнятся все поля, включая поле c компанией. При этом при таком запросе Hibernate выполнит LEFT OUTER JOIN с таблицей User(LEFT) и Company. Но как мы можем менять логику, которая будет выполняться при получении юзера, например, мы, может, хотим сделать INNER JOIN или вообще отдельно сделать запрос для получения информации о компании.

В ManyToOne аннотации есть возможность указать свойство optional, что означает optional=false = NOT NULL ограничение на поле company_id в таблице USER.Т.е. у каждого юзера обязательно будет компания, к которой он привязан, в таком случае вызовется уже INNER JOIN! В чем плюсы ? В производительности, т.к. когда бы делали LEFT JOIN, то из таблицы забирались все пользователи, у которых даже могло не быть компании, а в случае INNER JOIN те юзеры, у которых компания будет отсутствовать будут отброшены сразу же, что даст буст по производительности.

![[InputFiles/image 101 3.png|image 101 3.png]]

И есть еще одно свойство fetch, которая по умолчанию является EAGER для обычных полей и LAZY для полей, которые являются коллекциями.

![[InputFiles/image 102 3.png|image 102 3.png]]

Если мы сейчас установим fetch = FetchType.LAZY и выполним запрос get(), то выполнится только 1 запрос на получению user, при этом никаких JOIN происходить не будет. Что будет внутри user? Видим, что поля заполнены, а на месте company не null, а какой-то объекта типа CompanyHibernateProxy, который очень похож на наш класс Company, но в нем есть доп поле hibernate_interceptor. Но если мы потом у полученного user вызовем user.getCompany().getName()(просто любое поле попросим у компании), то уже выполнится доп запрос SELECT с получением информации о компании заполнению этой информации в объект типа Company.

![[InputFiles/image 103 3.png|image 103 3.png]]

Резюме:

У ManyToOne аннотации есть fetchType

1. FetchType.EAGER - жадная стратегия. При получении сущности user, будет вызываться получение и сущности из foreign key(company)
    1. optional=true вызовется LEFT OUTER JOIN
    2. optional=faslse вызовется INNER JOIN, что может быть эффективнее
2. FetchType.LAZY - ленивая стратегия. При получении user не будет происходить JOIN, чтобы сразу же заполнить поле из foreign key(company). Только если мы у company вызовем геттер какого-нибудь поля, то вызовется дополнительный SELECT для получения информации о company. При этом если просто сделаем user.getCompany(), то просто в качестве компани будет тот CompanyHibernateProxy.

### 4. 003 Hibernate Proxy

Как Hibernate использует proxy для ленивой инициализации?

Есть 2 варианта создания Proxy

1. Dynamic
    
    Для Dynamic мы используем готовое средство java для создания прокси - Proxy. Обработчик запросов - обработчик вызовов.
    
    ![[InputFiles/image 104 3.png|image 104 3.png]]
    
    ```Java
    @Test
        void testDynamic() {
            Company company = new Company();
            Proxy.newProxyInstance(company.getClass().getClassLoader(), company.getClass().getInterfaces(),
                    (InvocationHandler) (proxy, method, args) -> {
                        //proxy - прокси объект, который мы отдали пользователю и он на нем вызвал метод
                        //method - метод, который вызвал пользователь
                        //args - аргументы метода
                        //и уже можем решать, что делать с методом, например, вызвать на релаьном объекте или нет
                        return method.invoke(company, args);
                    });
    
        }
    ```
    
2. Extends - так поступает Hibernate, он наследует от нашего основного класса свой класс, создает поле типа ButyBuddyInterceptor и реализует несколько интерфейсов:
    
    ![[InputFiles/image 105 3.png|image 105 3.png]]
    
    ButyBuddyInterceptor основной его метод - это intercept, который полностью похож на метод из Dynamic Proxy и так же принимает объект прокси, метод и аргуметы.
    
    ![[InputFiles/image 106 3.png|image 106 3.png]]
    
    ![[InputFiles/image 107 3.png|image 107 3.png]]
    
    Кажый прокси объект хранит сессию, так как потом через этот прокси при вызове на нем метода с получением каких-то полей надо будет через эту сессию реально сходить в БД и получить данные! И это важно, поскольку если мы закроем сессию, то можем столкнуться с проблемой LAZY INITIALIZATION, т.к. сессия будет закрыта, а данные мы запросили, но уже в БД сгонять не может из-за закрытой сессии.
    
    Но важно еще отметить то, что у нас уже есть id нашей компании, т.к. она пришла, когда мы получали информацию о юзере, поэтому, если вызвать company.getId(), то запроса в БД не будет и инициализации company тоже, просто мы в прокси объекте храним id и вернем его. А вот если уже был вызван метод company.getName(), то запрос в БД будет, так как в proxy у нас нет такой информации и как раз, используя тот id, который мы сохранили в proxy объекте мы можем получить нужную нам информацию.
    
    Когда мы вызываем company.getName() вызывается метод intercept у нашего объекта byteBuddyInterceptor, затем метода invoke, в котором происходит определение того, что за метод мы вызвали, и внутри invoke есть куча if else, которые определяют требуется ли сходить в БД и инициализировать наш прокси объект данными из БД или нет.
    
    Вот как раз if else как пример того, что проверяет что за имя у метода и если это был какой-то метод, спецефичный для нашего прокси, а не сущности, которая стоит за прокси, то инициализировать сущность не надо
    
    ![[InputFiles/image 108 3.png|image 108 3.png]]
    
    И если не один из этих if не сработал, то возвращается INVOKE_IMPLEMENTATION(ну что-то типо флага, что нужно все-таки инициализировать наш объект и идти в БД)
    
    ![[InputFiles/image 109 3.png|image 109 3.png]]
    
    С помощью getImplementation должны получить объект, внутри него вызывается метод initialize() в котором проверяется не инициализирован ли уже объект, не равна ли null сессия(как раз та ошибка, о которой мы говорили ранее о LazyInitialization) и другие проверки на валидность сохраненной сессии. Все это происходит внутри объекта byteBuddyInterceptor, в котором хранится и сессия и вот эти все методы вызываются и есть поле Object target - которое будет проинициализировано нашим объектом company, если все-таки нужно будет инициализировать сущность.
    
    ![[InputFiles/image 110 3.png|image 110 3.png]]
    
    ![[InputFiles/image 111 3.png|image 111 3.png]]
    
    Вот некоторые поля byteBuddyInterceptor. Object target - как раз будет хранится наша сущность, т.е. у нас до конца в User в поле company будет хранится ссылка на byteBuddyInterceptor, а вот внутри byteBuddyInterceptor будет уже хранится наш реальный объект.
    
    ![[InputFiles/image 112 3.png|image 112 3.png]]
    
    Мы можем получить все-таки и сам объект, который лежит внутри Proxy, используя метод Hibernate.unproxy(user.getCompany()), передав туда компани. Тогда внутри этот метод проверит, действительно ли мы передали Proxy(для этого проверит есть ли интерфейс нужный у переданного объекта - проверит с помощью instanceof оператора) и если есть, то просто получим поле target перед этим вызвав опять метод initialize(), который его проициниализирует(либо, если уже проинициализированное поле, то ничего делать не будет и вернет сразу target).
    

### 4. 004 Cascade types

Разберем последнее свойство, которое есть внутри аннотации ManyToOne - это CascadeType. Как видим - они очень похожи на жизненный цикл нашей сущности и это не просто так.

![[InputFiles/image 113 3.png|image 113 3.png]]

Мы можем задать массив того, что будет происходить с состоянием company, когда наш user переходит в какое-то из своих состояний. Например, если мы получили юзера, он попал в persistanceContext, вместе с ним туда попала и компания(если ее тоже запросили, либо fetchType=FetchType.EAGER), затем мы вызвали session.evict(user) - т.е. удалили user из persistanceContext, то company не удалится, но, если мы поставим в @ManyToOne(cascade=CascadeType.DETACH), то в таком случае компания так же удалится из persistanceContext. И аналогично с другими видами, но нельзя забывать про parent-child. Главной таблицей является все-таки таблица компаний, поскольку именно юзеры на нее ссылаются. Поэтому, например, ставить @ManyToOne(cascade=CascadeType.PERSIST), чтобы при сохранении юзера сохранялась и компания - неправильно, поскольку в первую очередь должна существовать компания в таблице компаний, а потом уже только User сохраняется, который ссылается на компанию. Дальше мы пройдем аннотацию @OneToMany, которая будет уже у company и вот там уже можно такое настроить, чтоб при сохранении компании сохранились и юзеры, которые на нее ссылаются - это все-таки логичнее. И аннотации принимают массив CascadeType, поэтому мы можем сразу много их передавать:

![[InputFiles/image 114 3.png|image 114 3.png]]

На скрине выше мы решили проигнорировать то правило с parent-child и все равно попытаться сохранить user - возникает исключение!

Но мы можем костыльно обойти это, если в cascade установить ALL. Тогда все-таки Hibernate самостоятельно сначала сохранить Company, а потом уже user.

![[InputFiles/image 115 3.png|image 115 3.png]]

![[InputFiles/image 116 3.png|image 116 3.png]]

В общем, лучше использовать cascade над главной сущностью, в нашем случае у company и аннотация OneToMany, которую пройдем на следующем уроке.

### 4. 005 OneToMany

Первое слово относится к сущности, в которой мы исппользуем аннотацию, а второе - к полю в этой сущности. Так OneToMany в классе company One - относится к компании, а Many - к полю List< User> users:

![[InputFiles/image 117 3.png|image 117 3.png]]

И для реализации OneToMany есть 2 варианта:

1. Используем аннотацию JoinColumn(name=”название столбца по которому соединяем в таблице users”). Т.е. для связи используем БДшные столбцы
    
    ![[InputFiles/image 118 3.png|image 118 3.png]]
    
2. Т.к. мы в User уже определили ManyToOne, то можем связь настроить не на основе столбцов из БД, а на основе полей наших сущностей. Для этого используем mappedBy свойство и указываем название поля в классе User, через которое связаны наши сущности User и Company
    
    ![[InputFiles/image 119 3.png|image 119 3.png]]
    
    В таком случае нам уже не нужно использовать JoinColumn, так как мы связь настроили на основе полей, а не столбцов из БД.
    

Второй вариант используется чаще всего!

Теперь попробуем получить компанию и посмотреть, что будет внутри нее

![[InputFiles/image 120 3.png|image 120 3.png]]

Вызвав прошлый код мы увидим, что в company users поле заполнено каким-то PersistentBag - это аналоги прокси, который мы проходили только не для одной сущности, а для коллекции. Это именно для Lazy сущностей, т.к. по умолчанию для OneToMany FetchType=LAZY. Для коллекции это оправдано, потом что может быть сотни или тысячи элементов в коллекции.

![[InputFiles/image 121 3.png|image 121 3.png]]

А теперь поменяем тип коллекции на Set< User> users. Тогда произойдет stackOverFlow! Это потому что произошло зацикливание. Для добавления User в Set нужно у него посчитать equals и hashCode, но в User есть поле Company, которое так же участвует в hashCode и equals, поскольку мы не исключали это поле, поэтому для Company вычисляется hashCode, но для вычисления hashCode компани нужен hashCode списка юзеров и т.д. Поэтому мы должны исключить из Company поле users для ToString(т.к. дебаг пользуется этим методом, поэтому тоже зацикливание произойдет) и EqualsAndHashCode

![[InputFiles/image 122 3.png|image 122 3.png]]

И теперь видим PersistentSet(size стал 2, если вдруг ты заметил, это потому что он добавил еще одного пользователя)

![[InputFiles/image 123 3.png|image 123 3.png]]

### 4. 006 Cascade types with collections

Если сделать FetchType.EAGER для коллекции, то будет вызываться LEFT OUTER JOIN и лучше так не делать, поскольку если есть еще какие-то зависимости, то опять будет LEFT JOIN и мы получим очень много записей из БД, что плохо.

Изменим класс Company и посмотрим, как он будет себя вести, если мы сделаем cascadeType.ALL

```Java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
//@Table(name = "company") //Но эта аннотация необязательна, так как постгрес не чувствителен к регистру таблиц
public class Company {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Integer id;

    private String name;

    @Builder.Default //хотим, чтобы если не устанавливается поле через билдер, то использовалось то, чем мы инициализировали
    @OneToMany(mappedBy = "company", cascade = CascadeType.ALL)
    private List< User> users = new ArrayList<>();
    
    //полезно именно через метод добавлять пользователя, так как удобно можем установить ссылку на company для user
    public void addUser(User user) {
        users.add(user);
        user.setCompany(this);
    }
}
```

Если мы создадим компанию и добавлим туда пользователя, а потом сохраним компанию, то пользователь так же сохранится в БД. Сначала сохранится компания, а потом пользователь - это из-за того, что CascadeType=ALL. Если не выбирать CascadeType=ALL, то сохранится только компания. Аналогично будет и с другими состояниями, т.е. объекты users всегда будут повторять то состояние, в котором находится компания.

![[InputFiles/image 124 3.png|image 124 3.png]]

При удалении лучше явно самим вызывать метод get для получения компании, а потом передачи ее для удаления. Так мы удостовериваемся, что такая сущность реально есть в БД и ее нужно удалить. В случае удаления удаляется и компания и пользователи.

![[InputFiles/image 125 3.png|image 125 3.png]]

Но лучше всего использовать ON DELETE CASCADE на уровне БД, чтобы если удалили компанию, то пользователей, ссылающихся на эту компанию удаляла БД, а не Hibernate.

### 4. 007 Entity equals and hashCode

Не так давно из-за зацикленности сущностей Company и User мы столкнулись со stackOverflow при использовании Set коллекции(объяснение почему произошла зацикленность где-то в прошлом уроке). Чтобы обойти мы решили просто исключить то поле из-за которого произошла зацикленность - это достаточно оптимальный вариант для ToString метода, но вот для HashCode хочется чего-то другого. Например, мы могли бы использовать primary key = id у наших сущностей, но это не работает, поскольку, когда мы добавляем в коллекцию пользователей, которые не находятся в БД, то id у них null и тогда при добавлении двух пользователей у них одинаковый id=null, а, следовательно, и хешкод, поэтому лучше найти какое-нибудь поле в нашей сущности, которое является достаточно натуральным(в том смысле, что в целом оно уникально у сущностей). Например, у компании это ее название, поэтому мы можем использовать название компании в качестве поля для hashCode. Чтобы использовать только это поле используется @EqualsAndHashCode(of=”название поля”)

![[InputFiles/image 126 3.png|image 126 3.png]]

Для user можем использовать username, который так же является уникальным

![[InputFiles/image 127 3.png|image 127 3.png]]

Ограничение на поле для использования в хешкод - это not NULL и UNIQUE.

![[InputFiles/image 128 3.png|image 128 3.png]]

Но, что делать, если такого поля нет? Тогда нет варианта, кроме как использовать все поля, кроме тех, которые зацикливаются из-за связей.

### 4. 008 PersistentCollection

Как Hibernate лениво инициализирует наши коллекции?

Снова смотрим типы в Hibernate в данном случае есть базовый класс CollectionType и от него множество реализаций коллекций. И внутри этих коллекций есть метод instantiate, который создает Persistent вариант коллекции во внутрь этого объекта передает сессию.

И все эти Persistent коллекции наследуются от базового класса AbstractPersistentCollection, который очень похож по структуру на тот объект прокси Hibernate, который мы до этого обсуждали.

![[InputFiles/image 129 3.png|image 129 3.png]]

И внутри реализаций этого абстрактого класса добавляются как раз поле, которое будет соотвестоввать реальной коллекции, которая и будет в будущем проинициализирована. Она не будет проинициализирована до тех пор, пока мы не обратимся в сущности к коллекции.

![[InputFiles/image 130 3.png|image 130 3.png]]

Но мы можем явно попросить Hibernate проициализировать эту коллекцию, попросив проинициализировать прокси, для этого используется метод Hibernate.initialize(сюда передается прокси объект для инициализации(как раз коллекция, т.к. мы видели, что при получении из БД company в поле users сохранятся прокси объект).

![[InputFiles/image 131 3.png|image 131 3.png]]

Но на практике лучше избегать таких методов для инициализации прокси.

### 4. 009 LazyInitializationException

Мы многое узнали про то, как Hibernate работает с прокси объектами, а теперь узнаем, с какими проблемами мы можем столкнуться.

Если мы попытаемся проиницализировать транзитивный Lazy объект(даже косвенно, просто попытавшись получить его поле или вызвав на нем какой-то метод) ВНЕ сессии, в которой мы получили основной объект, то произойдет LazyInitializationException. Вот пример кода, когда он произойдет: Мы сначала получили компанию в одной сессии, а как мы знаем, поле List< User> users будет проицинаилизировано прокси коллекцией PersistentList, внутри которой хранится сессия, в которой создалась прокси. Потом мы вышли из try with resource сессия закрылась и после этого мы вызвали company.getUsers(), тогда этот вызов перешел на прокси коллекцию, она поняла, что пора проинициализировать внутреннию коллекцию и вызвала соответствующий метод, но, т.к. в ней хранилась сессия, которую уже закрыли, то произошло исключение.

![[InputFiles/image 132 3.png|image 132 3.png]]

Как решают такую проблему в реальных приложениях ? Открывают сессию на уровне сервисом и тогда к какому бы DAO мы не обращались на уровне сервисов мы общались бы через одну и ту же открытую сессию и все было бы хорошо. Т.е. когда Servlet обращается к Service, то смотрим, открыта ли какая-то сессия, если да, то используем ее, если нет, то открываем новую. Но что, если мы хотим поработать с данными на уровне Servlet или Listener. Тогда в таком случае, мы используем паттерн DTO. Т.е. мы на уровне Service получили сущность и потом из этой сущности получили все нужные поля и создали объект DTO, который соотвествует сущности и уже этот объект возвращаем на уровень Servlet. И тогда уже с этим DTO объектом можно безопасно работать и использовать любые геттеры и сеттеры поскольку он не связан ни с каким Hibernate прокси объектом.

![[InputFiles/image 133 3.png|image 133 3.png]]

Так же в Hibernate есть возможность сразу получить прокси объект, тогда никакого вызова в БД не будет, но при этом мы сможем пользоваться этим прокси объектом и в какой-то момент, когда мы поймем, что нужно его проинициализировать, то он проинициализируется. ДЛя этого используется метод Company company = session.getReference(Company.class, id); Но в таком случае, если мы снова попытаемся проинициализировать объект в прокси, когда сессия будет закрыта, то LazyInitializationException.

### 4. 010 OrphanRemoval

[](https://www.notion.soundefined)

Нам осталось рассмотреть последнее свойство в OneToMany OrphanRemoval.

![[InputFiles/image 134 3.png|image 134 3.png]]

Если orhpanRemoval = true, то тогда при удалении из коллекции пользователя, т.е. users.remove(user) этот пользователь так же удалится и в БД, т.е. вызовется DELETE FROM users where id = ?; Если же orpanRemoval = false, то когда мы удалим пользователя из коллекции пользователь из БД он не будет удален.

### 4. 011 OneToOne. Primary key

Чтобы реализовать OneToOne обязательно foreignKey должен быть UNIQUE, тогда в такой таблице всегда будет каждая запись ссылаться строго на 1 уникальную запись из другой таблицы.

Для реализации этого есть два основных подхода.

Первый вариант нужно использовать только если вторым вариантом сделать нельзяю.

Первый вариант - primary key является еще и foreign key - минусом является то, что мы теперь не можем использовать автогенерированные ключи для primary key, т.к. теперь нужно ссылаться на другую таблицу.

Реализация первого варианта:

![[InputFiles/image 135 3.png|image 135 3.png]]

Для связи в OneToOne можем использовать снова JoinColumn - его можно каждый раз воспринимать просто как название столбца с foreignKey.

![[InputFiles/image 136 3.png|image 136 3.png]]

Есть еще аннотация, которая делает то, что мы сделали до этого сами явно. Т.е. ей мы просто сказали, что наш первичный ключ является и foreign key, поэтому по нему и делается связь.

![[InputFiles/image 137 3.png|image 137 3.png]]

И теперь в User добавляем так же аннотацию OneToOne и ссылаемся на Profile с помощью mappedBy. Почему mappedBy в User? Аналогично с company, в company таблице не было никакого foreignKey, на эту таблицу ссылались, значит, она главная, аналогично и тут в user нет foreignKEy на Profile, а наобороо profile ссылается на user, поэтому user главнее и поэтому в нем используется mappedBy. Поле profile в классе User:

![[InputFiles/image 138 3.png|image 138 3.png]]

Правило: ставить cascadeType только в parent сущностях для других полей. Т.е. в Company есть поле List < User> users над ним мы можем ставить cascadeType и так же теперь в user есть поле Profile, user главнее profile, поэтому можем ставить над полем profile cascadeType, чтобы, например, при сохранении пользователя сохранялся и его профайл.

Чтобы не сталкиваться с ошибкой stackOverFlow не забываем исключать поля для связи при ToString

![[InputFiles/image 139 3.png|image 139 3.png]]

### 4. 012 OneToOne. Foreign key

На прошлом занятии мы реализовали первый вариант реализации OneToOne. Но он очень неудобный(в конспекте я не показал, но там много всяких проблем с тем, что нужно следить за ключами в Profile именно в java приложении, а не БД).

Поэтому рассмотрим более удобный вариант реализации OneToOne.

Для OneToOne fetchType.EAGER.

Переделываем табличку profile:

![[InputFiles/image 140 3.png|image 140 3.png]]

Класс Profile теперь выглядит следующим образом:

```Java
@Data
@Entity
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Profile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne
    @JoinColumn(name = "user_id")
    private User user;

    private String street;

    private String language;

    public void setUser(User user) {
        user.setProfile(this);
        this.user = user;
    }
}
```

В User у нас CascadeType.ALL, поэтому если мы установим profile в user, то при session.save(user) сохранится и profile.

![[InputFiles/image 141 3.png|image 141 3.png]]

session.save(user) сохранится и profile

![[InputFiles/image 142 3.png|image 142 3.png]]

Особенность работы с OneToOne. По умолчанию у нас fetchType.EAGER, но, даже если мы изменим его на LAZY - это не повлияет и все равно сразу же при получении user будет получен Profile, а не прокси Profile. Почему так? Потому что у User нет foreign key, по которому он может найти Profile, который с ним связан(да его даже может не быть). Допустим, Hibernate решает создавать прокси для Profile или нет при загрузке юзера. Но он не знает, есть ли Profile, у которого ключ ссылается на текущего юзера, т.е. Profile может просто не существовать, ему нужно проверить, существует ли он, а проверка по сути представляет собой получение Profile в таблице profiles и уже нет смысла в прокси, поскольку мы уже сделали запрос в БД, чтобы проверить существование Profile.

Единственный вариант реализовать OneToOne с LAZY инициализацией - это указать optional = false(т.е. сущность обязано быть в БД).

![[InputFiles/image 143 3.png|image 143 3.png]]

Но даже после этого у нас все равно сразу выполнится запрос за получению Profile и тогда такой вывод:

![[InputFiles/image 144 3.png|image 144 3.png]]

Т.е. если мы испльзуем отдельный первичный ключ для таблицы profile при реализации one to one, то невозможно сделать lazy инициализация у user при загрузке profile. Однако, мы можем сделать у Profile LAZE инициализацию для User. Т.е. загрузили profile, а user не загрузится из-за lazy.

![[InputFiles/image 145 3.png|image 145 3.png]]

### 4. 013 ManyToMany

У одного пользователя может быть много чатов, так и в одном чате может быть много пользователей - это ManyToMany.

![[InputFiles/image 146 3.png|image 146 3.png]]

![[InputFiles/image 147 3.png|image 147 3.png]]

На практике встречается реже всех, так как редко, когда есть таблица, в которой просто есть ссылки на 2 другие таблицы. Обычно это самостоятельная сущность, например, в чате может хранится иконка чата, когда пользователь вступил в чат, когда чат был создан и куча всего такого и тогда уже мы используем синтетический первичный ключ, а не смесь двух foreignKey.

Класс Chat

![[InputFiles/image 148 3.png|image 148 3.png]]

Класс User. JoinTable - по аналогии c JoinColumn, только мы сейчас же присоединяем не колонку, а целую таблицу, поэтому и JoinColumn.

![[InputFiles/image 149 3.png|image 149 3.png]]

Никаких CASCADE над ManyToMany не ставится, поскольку не имеет смысла удалять все чаты, если удалился юзер, либо удалять всех юзеров, если удалился какой-нибудь чат.

Метод внутри User:

![[InputFiles/image 150 3.png|image 150 3.png]]

Сохранение чата: Сначала произойдет сохранение chat, а потом, когда сессия flush(), то еще и добавится запись в users_chat

![[InputFiles/image 151 3.png|image 151 3.png]]

Но это сработало только потому что мы добавили chat в user классе. Там где mappedBy - там только readOnly.

![[InputFiles/image 152 3.png|image 152 3.png]]

А чтобы удалить записи из таблицы users_chat нам нужно получить пользователя из БД, а потом очистить у него поле Set< Chat> chats.

![[InputFiles/image 153 3.png|image 153 3.png]]

Итого: если хотим работать с таблицей users_chat, то делаем это через user.getChats(), т.е. через то поле, которое в User таблице ManyToMany.

### 4. 014 ManyToMany. Separate entity

На практике почти не встречается, чтобы в проекте была таблица, которая хранит только 2 ключа, как мы делали выше при реализации ManyToMany. Обычно есть еще множество других полей и поэтому такая таблица становится полноценной сущностью и у нее уже имеется PRIMARY KEY. Переделаем таблицу users_chat:

![[InputFiles/image 154 3.png|image 154 3.png]]

Тогда для такой сущности мы создаем полноценный класс, а для связи используем ManyToOne аннотации, т.к. другие и не подходят, OneToOne очевидно, ManyToMany - оканчивается на Many, значит, должен быть массив, что неверно, OneToMany аналогично. Для User не использовался JoinColumn, чтобы показать, что Hibernate по умолчанию возьмет название поля, т.е. user и добавит к нему _id. А для chat мы сделали это же только явно, использовав JoinColumn.

![[InputFiles/image 155 3.png|image 155 3.png]]

В User теперь вот так:

![[InputFiles/image 156 3.png|image 156 3.png]]

В Chat теперь вот так:

![[InputFiles/image 157 3.png|image 157 3.png]]

И чтобы в java классах у нас все соотвествовало БД добавляем 2 соотвествующих метода.

![[InputFiles/image 158 3.png|image 158 3.png]]

Сохранение UserChat в БД

![[InputFiles/image 159 3.png|image 159 3.png]]

### Мои выводы и какие-то советы

Аннотация JoinColumn используется для той сущности, в которой в БД есть foreignKey на другую сущность. В JoinColumn мы и указываем это. Тогда та сущность, в которой нет ForeignKey, но которая тоже с нами в связи испльзует уже mappedBy!

Когда реализуем ManyToMany хоть связь и бидирикшенл, но мы должны выбрать одну из сторон как owningside. Та, которая owning-side должна иметь JoinTable, а значит вторая сторона использует mappeBy. Тут полная аналогия с тем, что писалось раньше. Одна сторона имет JoinColumn, а другая mappedBy. Как выбрать owning-side? На самом деле разницы нет, любая сторона может быть owning-side, но можно просто подумать, какую сущность ты будешь чаще получать из БД и с ней работать - такая и будет owning-side.

Краткое резюме:

Один ко многим:

1. ManyToOne, JoinColumn
2. OneToMany(mappedBY)

Один к одному:

1. OneToOne, JoinColumn(название колонки, по которой соединянем) - та сущность, в которой есть foreignKey
2. OneToOne(mappedBy)

Многие ко кногим

1. Создаем доп таблицу и в ней:
2. ManyToOne, JoinColumn
3. ManyToOne. JoinColumn
4. В сущности 1 пишем OneToMany(mappedBy=поле в доп таблице)
5. В сущности 2 пишем OneToMany(mappedBy=поле в доп таблице)

### 4. 015 Collection performance

![[InputFiles/image 160 3.png|image 160 3.png]]

Лучше использовать List или Collection интерфейс для коллекций, а не Set, поскольку в случае Set каждый раз делаются большие запросы.

### 4. 016 ElementCollection

Создадим таблицу company_locale, которая будет хранить описание компаний на разных языках. Но мы не хотим создавать полноценную сущность в нашем java приложении под такую таблицу, т.к. она очень несамостоятельная и очень сильно зависит о компании. Для этого можно использовать Embedded компоненты.

![[InputFiles/image 161 3.png|image 161 3.png]]

Создаем embeded компонент:

![[InputFiles/image 162 3.png|image 162 3.png]]

В классе company заводим поле такого типа и указываем с помощью аннотации CollectionTable из какой таблицы брать значения и по какому ключу.

![[InputFiles/image 163 3.png|image 163 3.png]]

И теперь при добавлении значений в List< LocaleInfo> эти сущности будут добавлены в таблицу company_locale.

![[InputFiles/image 164 3.png|image 164 3.png]]

Если, например, мы хотим только получать значения description из таблицы, то добавляем аннотацию Column и указываем, что за столбец мы хотим получать. И важно, что мы только считывать можем в таком случае, т.к. там еще нужен lang.

![[InputFiles/image 165 2.png|image 165 2.png]]

### 4. 017 Collection ordering

Хотим получить отсортированный список пользователей в нашем java приложении. Есть 2 варианта - это отсортировать на уровне БД, т.е. использовать при запросе ORDERBY, либо отсортировать уже на стороне java приложения какой-нибудь сортировкой.

1-ый вариант(ORDERBY):

Для этого используется аннотация OrderBy, но она есть как в пакете javax.persistence так и в org.hibernate.annotations. Разница в том, что org.hibernate использует SQL(может использовать и HQL), а javax.persistance HQL

![[InputFiles/image 166 2.png|image 166 2.png]]

Используем org.hibernate.annotations, следовательно, передаем SQL код, но только окончание того, что мы пишем после ORDER BY, т.е. столбцы, по которым сортируем и в данном случае, т.к. SQL, то пишем имена СТОЛБЦОВ. И это важно, т.к. когда будем использовать ORDER BY из javax.persistence, то там уже HQL И будем использовать названия ПОЛЕЙ нашего класса.

![[image 167.png]]

Использование javax.persistence и HQL и такая же сортировка по username и lastname, но т.к. в User у нас нет поля lastname, а есть поле personalInfo, внутри уже которого lastname, то получилось personalInfo.lastname

![[image 168.png]]

Но мы можем заметить, что users мы храним в HashSet, но как тогда обеспечивается порядок, ведь HashSet не гарантирует порядок, нужно использовать какой-нибудь LinkedHashSet? А дело в том, что Hibernate использует свои коллекции, как мы говорили ранее и в случае Set используется PersistentSet, внутри которого используется как раз LinkedHashSet, поэтому и получилось, что сохранились пользователи все-таки в отсортированном виде.

Теперь рассмотрим второй вариант, когда сортировка в java. Тогда придется все-таки отказаться от HashSet и использовать TreeSet, который как раз хранит данные отсортированными. Но для сортировки тип для хранения должен реализовывать интерфейс Comparable, поэтому наш класс User должен его реализовать.

![[image 169.png]]

![[image 170.png]]

![[image 171.png]]

Если мы хотим, чтобы под капотом Hibernate использовал PersistentSortedSet, то должны наше поле users, объявить как SortedSet< User>, но тогда в таком случае аннотация SortNatural обязательна.

![[image 172.png]]

### 4. 018 Maps in mappings

Как использовать Map для маппинга наших сущностей?

![[image 173.png]]

Но есть и другие аннотации, например, если мы хотим в качестве ключа нашей сущности использовать целый класс, а не просто поле, тогда используем MapKeyClass. Но в реальных приложениях используется чаще всего MapKey.

![[image 174.png]]

Еще может быть полезно MapKeyColumn, но я не стал конспектить(это нужно для Embeded сущностей при использовании Map).

### 5. 001 In-Memory databases. H2

Как в реальных приложениях запускают тесты на БД? Мы сейчас запускали тесты на реальной БД, что, наверное, не очень хорошо)

Вспомним, как мы работаем с БД с использованием СУБД - у нас данные хранятся на диске.

![[image 175.png]]

В случае же In-memory БД данные хранятся в памяти и только если попросить(причем не все in-memory БД могут) то сохранить данные на жесткий диск.

![[image 176.png]]

Минусом является то, что если не синхронизированы данные с жестким диском, то при отключении света или падении приложения мы потеряем все данные. Но плюсом является то, что т.к. данные в оперативной памяти, то получение таких данных в 10-ки раз быстрее получения из БД. Ну и т.к. в оперативной памяти, то сохраним мы можем в целом меньше данных, чем на жестком диске.

Для работы нам нужно подключить зависимость h2-maven

![[image 177.png]]

И теперь, чтобы при выполнении тестов они выполнялись на h2 БД, а не на постгрес мы должны создать еще один конфигурационный файлик для hibernate, но уже в директории resources наших тестов, тогда при выполнении тестов hibernate будет использовать именно этот конфигурационный файлик. Но так же нужно заменить там url, user, password, драйвер и все такое.

Сайт с документацией по использованию h2 БД [https://www.h2database.com/html/features.html#products_work_with](https://www.h2database.com/html/features.html#products_work_with)

С него мы и берем URL

![[image 178.png]]

Когда мы запускаем наши тесты нам нужно теперь еще и поднять БД в оперативной памяти, но откуда создадутся таблицы ? Есть 2 вариант - 1-ый это использование специальных средств миграции, например, liquibase. 2-ой - это использование генераторов. В hibernate есть встроенный генератор, который на основе созданных нами сущностей создаст DDL код для создания таблиц,но минусом является то, что на продакшене придется проверять, правильно ли hibernate все сгенерировал.

У генератора есть несколько вариантов:

![[image 179.png]]

- update - смотрим разницу между сущностью в java и готовой схемой(она, наверное, при первом запуске и создается где-то) и если такая разница есть, то изменяем существующий DDL
- create - просто накатываем новую DDL, на новую БД
- create-drop - как только мы закрываем созданную SessionFactory мы дропаем всю БД, которую создали
- validate - проверка соотвествия схемы БД существующей с маппингом сущностей.

В нашем случае будем использовать create.

И теперь получается при запуске тестов каждый раз будет создаваться новая БД с сущностями, а при окончании тестов дропаться. Но мы попытались запустить наши тесты и столкнулись с еще одним минусом h2 - т.к. это уже не постгрес, то не все типы из java могут быть смаплены на тип из h2, поэтому мы рассмотрим более хороший вариант, который используется в реальных приложениях.

Вот что происходит при запуске. Сначала дропаются БДшки, а потом генерируется DDL код для создания БД и в конце уже наши запросы в Бд.

![[image 180.png]]

![[image 181.png]]

### 5. 002 Docker. Testcontainers

Используем Docker, в котором поднимается БД, а также библиотеку Testcontainers, которая при запуске тестов поднимает докер с БД, а при окончании тушит докер контейнер. В реальных приложениях используется именно такой способ! Если захочется, то посмотришь видео потом, как это все настроить.

### 6. 001 MappedSuperclass

На этих уроках мы изучим маппинг полей при наследовании сущностей в Hibernate. Например, мы можем заметить, что у нас у многих сущностей есть поле Integer id. А почему бы нам не создать какой-то базовый класс и чтобы все другие сущности отнаследовали это поле от этого базового класса.

Создадим класс BaseEntity и параметризуем его, чтобы id могли быть как Integer так и Long

![[image 182.png]]

И теперь в User мы можем убрать поле id и просто отнаследоваться от BaseEntity.

![[image 183.png]]

Но такой вариант не совсем подходит, поскольку не все сущности будут иметь strategy=GEnerationType.IDENTITY. А этот способ только для таких сущностей. Поэтому обычнро создают интерфейс, который все сущности реализуют(потому что у всех сущностей обязан быть id, поэтому они спокойно его могут реализовать). И тогда мы можем работать с сущностями в DAO каком-нибудь на уровне интерфейса BaseEntity.

![[image 184.png]]

Но, если мы все-таки хотим какие-то поля сделать общими для сущностей, например, на практике очень часто возникают сущности Auditable, в которых сохранятся кем создана сущность и когда и чтобы в таких сущностях не дублировать поля мы можем как раз воспользоваться функционалам MappedSuperClass.

![[image 185.png]]

### 6. 002 Inheritance. TABLE_PER_CLASS

Стоит заметить, что в прошлом варианте тот базовый класс от которого мы наследовались(AuditableEntity или BaseEntity, до того, как превратился в интерфейс) - они не были сущностями Hibernate, т.е. для них не было отдельной табличке в БД. Но что, если мы хотим использовать наследование, когда и базовый класс и наследник являются сущностями. Например, есть базовый класс User - это entity, а мы хотим отнаследовать от него класс программист, который так же является пользователем, но при этом имеет какие-то дополнительные поля и этот класс программист тоже хотим, чтобы был entity. И в hibernate есть 3 различные стратегии маппинга при наследовании:

![[image 186.png]]

Table_PER_CLASS

Каждый из наследников - это отдельная таблица и все поля просто будут дублироваться.

Создадим отдельные классы для Manager и Programmer

![[image 187.png]]

![[image 188.png]]

Добавляем их в Configuration

![[image 189.png]]

И далее в User мы должны навесить аннотацию Inheritance и выбрать стратегию, в нашем случае - это TABLE_PER_CLASS

![[image 190.png]]

Так же, теперь у нас нет таблицы User, поэтому мы делаем класс абстрактным, а также удаляем аннотацию Builder.

![[image 191.png]]

Но это еще не все! Так же нам нужно теперь поле Id изменить strategy и заменить  
  
`@GeneratedValue(strategy = GenerationType.`_`IDENTITY`_`)` заменяем на `@GeneratedValue(strategy = GenerationType.SEQUENCE)` и придется добавить общий сиквенс для двух таблиц. Зачем так делать ? А затем, чтобы не было дубликатов id в двух таблицах. Ну т.е. если у нас отдельный сиквенс для каждой из таблиц, то в таблице programmer будет строка с id =1 и в таблице manager так же будет такая строка. Почему это неверно? Потому что у нас есть базовая сущность User и если мы захотим получить User с id=1, то теперь непонятно из какой таблицы его брать, поэтому и нужно, чтобы id было уникальным уже между всеми таблицами, а не только внутри нашей таблицы!

Сделаем вот такой get:

![[image 192.png]]

Получим такой SQL(просто обратились к таблице программистов и нашли по id).

![[image 193.png]]

А если при get будем указывать не конкретную таблицу, а общую сущность User.class.

![[image 194.png]]

Тогда произойдет UNION ALL двух запросов. Первый запрос сходит в таблицу программистов, а второй сходит в таблицу менеджеров и произойдет их объединение, причем как мы знаем, при UNION ALL должны совпадать полностью типы и количество полей, поэтому Hibernate дополнительно добавляет поля при запросе, чтобы следовать этому правилу и затем из этого объединения по ID находится нужный User, но, чтобы знать, из какой на самом деле таблицы был взят User, дополнительно добавляется в запросе поле clazz(0 - из программистов, 1 - из менеджеров). И потом по этому полю определяется из какой таблицы было взято.

Плюсы такой реализации:

Если нам нужен какой-то конкретный наследник, например, программист, то нам не нужно искать во всех таблицах наследников, а ищем в конкретной таблице.

Минусы:

Приходится дублировать общие поля в обоих таблицах. И если добавляется какое-то общее поле в User, то нам нужно будет изменять и таблицу programmers и таблицу managers.

Когда получаем весь список пользователей(т.е. обращаемся к базовому классу User), то используется UNION ALL

Нужен общий сиквенс для двух таблиц

### 6. 003 Inheritance. SINGLE_TABLE

Для всех наследников будет одна таблица. И в ней будут как общие поля, так и конкретные поля подклассов и + колонка type, по которой мы будем понимать к какому реально классу относится запись к программисту или менеджеру.

![[image 195.png]]

Реализация в коде вот такая:

![[image 196.png]]

И в подклассах мы с помощью аннотации определяем какое значение будет иметь поле type для нашего подкласса

![[image 197.png]]

Тогда создаается такая таблица, в котором объединение полей

![[image 198.png]]

А вот такой код будет сгенерирован при поиске программиста session.get(Programmer.class, 1L). Т.е. и id сравнится, и чтобы это было программистом сравнивается поле type.

![[image 199.png]]

А когда мы ищем не конкретную сущность, а просто user(session.get(User.class, 1L), то будет такой же SELECT как и выше, но не будет ограничения по type

И т.к. одна и та же таблица, то ID снова можно сделать IDENTITY

![[image 200.png]]

Минусы:

Одна таблица со всеми полями со всех наследников, что не дает нам делать ограничения на поля. Например, мы не можем поставить NOT NULL ограничение на какое-то поле, поскольку это поле будет NULL для всех других наследников, кроме одного, кому это поле реально принадлежит. Т.е. каждая запись может принадлежать какому-то из наследников и поэтому поля других наследников становятся не проиницализированы.

При поиске придется искать по всей таблице из всех наследников, тогда как при TABLE_PER_CLASS мы могли быстренько найти в нужной таблице человека, если знаем, что он принадлежит, например, программмистам.

Плюсы:

Не нужно делать UNION ALL двух таблиц.

Можем использовать IDENTITY ключи.

### 6. 004 Inheritance. JOINED

Создаем отдельные таблицы для каждой сущности. Для базового класса создается таблица, в которой колонки представляют собой общие поля, а для наследников мы создаем таблицы, в которых находятся дополнительные поля наследников. Т.е. такая схема больше похожа на java схему наследованиия. А также в наследниках будет user_id - который одновременно является и PRIMARY KEY и FOREIGN KEY на таблицу users.

![[image 201.png]]

Меняем тип наследования на JOINED в базовом классе.

![[image 202.png]]

В наследниках ставится аннотация PrimaryKeyJoinColumn, где мы говорим, что наш первичный ключ будет еще и FOREIGNKEY на таблицу User. name = это название нашего ключа в нашей таблице. А Id к нам попало за счет наследования от базового класса User.

![[image 203.png]]

Тогда при вставке, например, нового программиста будет происходить два инсерта, один со значениями общих полей в таблицу users, а второй уже со значением кастомных полей + id из users в таблицу programmers:

![[image 204.png]]

UPDATE и DELETE так же усложнятся, поскольку нужно менеджить сразу две таблицы.

При get программиста будет INNER JOIN, т.к. нужны поля как из общей таблицы, так и из таблицы программистов

![[image 205.png]]

А если мы при запросе не указываем, кто это программист или менеджер, т.е. session.get(User.class, 1L), то Hibernate делает LEFT JOIN для каждой из таблиц(INNER не используется, поскольку запись может быть как в таблице программистов так и в таблице менеджеров) и потом уже из этого общего объединения с помощью where пытается найти нужную запись и так же добавляет поле для определения из какой таблицы реально была забрана сущность.

Плюсы:

Данные нормализованы, в отличие от SINGLE_TABLE, т.к. у нас нет дублежа данных, т.е. у нас в каждой отдельной таблице хранятся единичные данные без дублирования информации.

Минусы:

Для вставки, удаления, обновления - все усложняется, поскольку нужно поддерживать сразу 2 таблицы

SELECT так же усложняется, поскольку нужно выполнять LEFT JOIN для каждой из дочерних таблиц.

### Итог по всем вариантам

Лучше всего использовать SINGLE_TABLE, поскольку в плане производительности это самый эффективный способ(скорее всего, потому что нет никаких JOIN запросов и по сути всегда работает с уже заджоиненной таблицой, хоть в ней и больше данных, но из-за быстрых алгоритмов поиска по id при B-tree индексах это похоже не так критично, как делать большие джоины таблиц). Но так же на практике бывает еще используют JOINED, и реже всего TABLE_PER_CLASS.

### 7. 001 HQL. Part 1

Нам не всегда хватает того функционала, который нам предоставляет hibernate. Иногда нам нужно сделать какие-то более сложные запросы, например, SELECT со множеством WHERE условий. И для этого в Hibernate есть 2 варианта - использование HQL и criteria API. HQL чаще используется.

JPQL - это спецификация, HQL - конкретная реализация, как в случае с JPA и Hibernate.

Один и тот же запрос, написанный на SQL(закомментирован) и на HQL. Основная разница в том, что в HQL мы работает с нашими сущностями, а не с таблицами в БД.

![[image 206.png]]

Результатом createQuery является объект типа Query.

![[image 207.png]]

И в нем довольно много методов, но основные из них - это getResultList(), возвращающий List<>, есть аналогичный ему метод .list().

iniqueResult / uniqueResultOptional- возвращает одну сущность, если наш запрос возвщарает 1 сущность или мы вызвали какой-нибудь INSERT, который возвращает количество всталенных записей.

setParameter методы на все случаи жизни(аналог с preparedStatement)ю

setMaxResult = LIMIT

setFirstResult = OFFSET

При вызове createQuery мы можем указать, какой тип ожидаем, чтобы при вызове list(), получимть List< User>, а не просто List объект.

![[image 208.png]]

Но имя Ivan мы явно не хотим хардкордить, а хотим заполнять его, как мы это делали в preparedStatement. Поэтому мы можем использовать либо именованные параметры, либо по позиции, как на скрине ниже. Для этого мы указываем знак вопрос и номер параметра, а потом через setParameter указываем номер параметра, который хотим вставить!

![[image 209.png]]

Но лучше использовать именованные параметры, поскольку если вдруг мы поменяем их порядок, то лучше бы, чтобы ничего не сломалось, а это как раз решит эту проблему!

![[image 210.png]]

Как сделать JOIN? Мы знаем, что оперируем теперь с нашими java сущностями и знаем, что в наших сущностях уже настроена связь через наличие полей в классах на другие сущности, ну так давайте их и использовать и при join просто обращаемся к полю сущности с которой хотим заджоиниться.

![[image 211.png]]

Но самое прикольное, что выше мы сделали явный джоин, а мы можем еще и обращаться напрямую к полям другой сущности, не делая JOIN и тогда hibernate сделает за нас этот jOIN, хотя мы явно не писали это!

![[image 212.png]]

Но явный джоин все-таки лучше! Поскольку явное, лучше неявного

### 7. 002 HQL. Part 2

Если у нас из раза в раз повторяется один и тот же запрос, то мы можем сделать именованный запрос, т.е. сохранить запрос и дать ему имя и потом переиспользовать(на практике не очень частая тема, поскольку HQL запросы могут быть очень большими).

Для этого над классом сущности пишем аннотацию NamedQuery. Мы пишем это именно над той сущностью, которую возвращает наш запрос. В данном случае запрос возвращает User, поэтому и аннотация над User.

![[image 213.png]]

Теперь создается не query, а namedQuery, где и указывается имя запроса, которое указали в аннотации.

![[image 214.png]]

Вместе с запросом можем задавать какие-то hint(доп информация). А так же есть flushMode - это важный параметр для запроса, который определяет в какой момент будет выполнен flush для нашего запроса. По умолчанию это AUTO. Т.е. в момент выполнения запроса, а точнее перед его выполнением будет выполнен метод flush. Т.е. нужно ли у сессии вызывать метод flush перед выполнением нашего запроса или нет. Если auto, то вызывается, если COMMIT, то вызовется вместе с COMMIT всей сессии. На практике чаще всего AUTO.

![[image 215.png]]

HQL так же позволяет создавать UPDATE,INSERT, DELETE запросы, которые возвращают количество обновленных данных, поэтому мы вызываем executeUpdate(), который нам известен еще с JDBC.

![[image 216.png]]

Но если мы укажем RETURNIN *, то можно получить и сами обновленные строки. Однако для этого нужно использовать уже нативный SQL, т.е. создаем уже createNativeQuery и внутрь уже передается SQL код, а не HQL! И это довольно часто используется, так как hibernate не всегда оптимален в плане запросов, однако офигенный плюс в том, что хоть мы для запроса и использовали SQL, однако hibernate все равно в состоянии смапить его результат на наши сущности!

![[image 217.png]]

### 003 HQL. Практика

Некоторые изменения в нашем коде. В классе User добавили связь OneToMany.

![[image 218.png]]

Payment - по сути таблица выплаты зарплаты. В ней указывается сколько выплатили и кому.

![[image 219.png]]

![[image 220.png]]

![[image 221.png]]

![[image 222.png]]

Неявный JOIN не работает с коллекциями.Когда мы просто пишем JOIN, то вызывется INNER JOIN, если нам нужен left то явно пишем LEFT JOIN.

![[image 223.png]]

![[image 224.png]]

![[image 225.png]]

![[image 226.png]]

![[image 227.png]]

### 7. 004 Criteria API

Мы изучим второй вид запросов, которые можно делать в Hibernate. Благодаря ним можно делать динамические запросы, например, с условиев where, где может быть произвольное количество критериев.

![[image 228.png]]

Через criteria строим наш каркас запроса, перед этим получим таблицу, с которой будем работать, вызвав метод from. А с помощью CriteriaBuilder получаем доступ ко всяким БДшным штукам, ну т.е. заполняем уже сам каркас нашего запроса. Причем можно заметить один сильный минус, это то, что чтобы получить значения поля, нам нужно передавать строку(я сейчас про строчку, где user.get(”personalInfo”).get(”firstname”)). Но есть специальный модуль, который генерирует статические поля для наших классов с такими строками и тогда мы будем обращаться к этим статическим полям, что сильно лучше, т.к. при использовании строк велик шанс ошибки.

![[image 229.png]]

Подключаем этот модуль:

![[image 230.png]]

Вот такие сущности для наших классов будут сгенированы:

![[image 231.png]]

И теперь мы можем обращаться к этим сущностям, а точнее к их статическим полям.

![[image 232.png]]

![[image 233.png]]

![[image 234.png]]

![[image 235.png]]

Вот так примерно выглядел бы динамический where. Чаще всего в функцию просто приходит объект DTO с множеством полей и мы через if проверяем установлено ли поле или нет и в зависимости от этого добавляем критерий или нет.

![[image 236.png]]

![[image 237.png]]

Так указывается тип JOIN:

![[image 238.png]]

для того, чтобы вернуть множество объектов используется multiselect, в который и передается то, что мы хотим вернуть.

![[image 239.png]]

Но multiSelect так же позволяет возвращать не просто список объектов, а замаппить результат на DTO класс, который мы написали

![[image 240.png]]

![[image 241.png]]

Единственное была ошибка, потому что теперь же мы возвращаем не массив объектов, а всего 1 объект DTO, поэтому и SELECT должен быть обычный.

![[image 242.png]]

![[image 243.png]]

![[image 244.png]]

### 7. 005 Querydsl. Настройка

Мы увидели, как непросто на самом деле писать запросы, используя Criteria API, поэтому есть библиотечки, которые помогают писать динамические запросы. Одна из таких - Querydsl.

![[image 245.png]]

Добавляем две зависимости, одна для добавления новых классов, а вторая для того, чтобы сгенерировалась новая модель.

![[image 246.png]]

Лучше посмотреть потом в интернете, как запустить все это, т.к. думаю, что все могло измениться.

### 7. 006 Querydsl. Практика

![[image 247.png]]

![[image 248.png]]

![[image 249.png]]

![[image 250.png]]

![[image 251.png]]

![[image 252.png]]

с DTO работать QueryDSL не умеет, поэтому используем tuple, который добавил querydsl.

![[image 253.png]]

![[image 254.png]]

### 7. 007 Querydsl. Filters

Создадим класс билдер для наших предикатов

![[image 255.png]]

Создадим DTO, в котором могли устанавливаться множество фильтров.

![[image 256.png]]

И теперь можем вот таким прикольным способом получать DSL, используя переданный фильтр.

![[image 257.png]]