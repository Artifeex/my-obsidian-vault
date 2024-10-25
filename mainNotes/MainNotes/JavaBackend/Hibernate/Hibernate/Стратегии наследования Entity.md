## Mapped Superclass

Mapped Superclass это класс от которого наследуются Entity, он может содержать аннотации JPA, однако сам такой класс не является Entity, ему не обязательно выполнять все требования установленные для Entity (например, он может не содержать первичного ключа). Такой класс не может использоваться в операциях EntityManager или Query. Такой класс должен быть отмечен аннотацией MappedSuperclass или соответственно описан в xml файле.
## Стратегии наследования маппинга

В JPA описаны три стратегии наследования маппинга (Inheritance Mapping Strategies), то есть как JPA будет работать с классами-наследниками Entity:

1) одна таблица на всю иерархию наследования (a single table per class hierarchy) — все entity, со всеми наследниками записываются в одну таблицу, для идентификации типа entity определяется специальная колонка “discriminator column”. Например, если есть entity Animals c классами-потомками Cats и Dogs, при такой стратегии все entity записываются в таблицу Animals, но при это имеют дополнительную колонку animalType в которую соответственно пишется значение «cat» или «dog».Минусом является то что в общей таблице, будут созданы все поля уникальные для каждого из классов-потомков, которые будет пусты для всех других классов-потомков. Например, в таблице animals окажется и скорость лазанья по дереву от cats и может ли пес приносить тапки от dogs, которые будут всегда иметь null для dog и cat соответственно.

Пример с кодом:

|   |
|---|
|@Entity(name="products")  <br>@Inheritance(strategy = InheritanceType.SINGLE_TABLE)  <br>@DiscriminatorColumn(name="product_type",  <br>  discriminatorType = DiscriminatorType.INTEGER)  <br>public class MyProduct {  <br>    // ...  <br>}|

  

Теперь подклассы можно разделять по @DiscriminatorValue с переданным значением для product_type:

|   |
|---|
|@Entity  <br>@DiscriminatorValue("1")  <br>public class Book extends MyProduct {  <br>    // ...  <br>}  <br>@Entity  <br>@DiscriminatorValue("2")  <br>public class Pen extends MyProduct {  <br>    // ...  <br>}|

  

Также можно использовать аннотацию Hibernate @DiscriminatorFormula:

|   |
|---|
|@Entity  <br>@Inheritance(strategy = InheritanceType.SINGLE_TABLE)  <br>@DiscriminatorFormula("case when author is not null then 1 else 2 end")  <br>public class MyProduct { ... }|

  

2) объединяющая стратегия (joined subclass strategy) — в этой стратегии каждый класс entity сохраняет данные в свою таблицу, но только уникальные колонки (не унаследованные от классов-предков) и первичный ключ, а все унаследованные колонки записываются в таблицы класса-предка, дополнительно устанавливается связь (relationships) между этими таблицами, например в случае классов Animals (см.выше), будут три таблицы animals, cats, dogs, причем в cats будет записана только ключ и скорость лазанья, в dogs — ключ и умеет ли пес приносить палку, а в animals все остальные данные cats и dogs c ссылкой на соответствующие таблицы. Минусом тут являются потери производительности от объединения таблиц (join) для любых операций.

Пример с кодом:

|   |
|---|
|@Entity  <br>@Inheritance(strategy = InheritanceType.JOINED)  <br>public class Animal {  <br>    @Id  <br>    private long animalId;  <br>    private String species;  <br>    // constructor, getters, setters  <br>}|

А подкласс можно просто объявить без дополнительных аннотаций:

|   |
|---|
|@Entity  <br>public class Pet extends Animal {  <br>    private String name;  <br>    // constructor, getters, setters  <br>}|

В обеих таблицах будет присутствовать animalId колонка.

3) одна таблица для каждого класса (table per concrete class strategy) — тут все просто каждый отдельный класс-наследник имеет свою таблицу, т.е. для cats и dogs (см.выше) все данные будут записываться просто в таблицы cats и dogs как если бы они вообще не имели общего суперкласса. Минусом является плохая поддержка полиморфизма (polymorphic relationships) и то что для выборки всех классов иерархии потребуются большое количество отдельных sql запросов или использование UNION запроса.

### Полиморфизм

В отличии от SQL в запросах JPQL есть автоматический полиморфизм, то есть каждый запрос к Entity возвращает не только объекты этого Entity, но также объекты всех его классов-потомков, независимо от стратегии наследования (например, запрос select * from Animal, вернет не только объекты Animal, но и объекты классов Cat и Dog, которые унаследованы от Animal). Чтобы исключить такое поведение используется функция TYPE в where условии (например select * from Animal a where TYPE(a) IN (Animal, Cat) уже не вернет объекты класса Dog).
