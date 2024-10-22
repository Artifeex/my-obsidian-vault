![[Pasted image 20240926111434.png]]
compare возвращает:
1. Положительное, если o1 > o2
2. Отрицательное, если o1 < o2
3. Ноль, если o1 = o2.
Возвращаемое значение можно запомнить, как как будто выполняется операция o1 - o2.
Интересным является то, что нам нужно реализовать только метод compare, а метод equals не нужен, т.к. когда мы пишем реализацию, то как мы знаем любой класс наследуется от Object и поэтому реализация для equals предоставится из Object.


Начиная с Java 8, интерфейс был значительно улучшен, и в нем появились **статические методы**, которые позволяют более легко и гибко создавать компараторы. Эти методы помогают избежать создания анонимных классов и упрощают код.

### Основные статические методы интерфейса `Comparator`:

1. **`comparing()`** - 
2. **`thenComparing()`**
3. **`naturalOrder()`**
4. **`reverseOrder()`**
5. **`nullsFirst()`**
6. **`nullsLast()``

### 1. `Comparator.comparing()`
Метод `comparing()` — это самый часто используемый способ создания компаратора. Он позволяет создать компаратор, основанный на значении свойства объекта. Обычно используется с **лямбда-выражениями** или методами-референсами.

Пример:
```java
Comparator<Employee> byName = Comparator.comparing(Employee::getName);
```
Здесь создается компаратор для объектов класса `Employee`, который будет сравнивать их по полю `name`.

Если необходимо сравнивать объекты по нескольким полям, можно добавлять дополнительные уровни сравнения с помощью метода `thenComparing()`.

Пример:
```java
Comparator<Employee> byNameThenAge = Comparator.comparing(Employee::getName)
                                                .thenComparing(Employee::getAge);
```
Этот компаратор сначала сравнит сотрудников по имени, а если имена одинаковы, то по возрасту.

Внутрь должна быть передана функция, преобразующая объект в объект, который реализует интерфейс Comparable, ведь нужно потом сравнивать их.

### 2. `Comparator.thenComparing()`
Метод `thenComparing()` используется для указания дополнительного критерия сравнения, если предыдущие критерии оказались равными. Он применяется после вызова основного метода сравнения, как показано выше.

Пример:
```java
Comparator<Employee> byNameThenAge = Comparator.comparing(Employee::getName)
                                                .thenComparing(Employee::getAge);
```
Этот компаратор сначала сортирует по имени, а если имена одинаковы — по возрасту.

### 3. `Comparator.naturalOrder()`
Метод `naturalOrder()` возвращает компаратор, который сравнивает элементы в **естественном порядке**. Этот метод подходит для классов, которые реализуют интерфейс `Comparable`.

Пример:
```java
Comparator<Integer> naturalOrderComparator = Comparator.naturalOrder();
List<Integer> numbers = Arrays.asList(3, 1, 2);
numbers.sort(naturalOrderComparator);  // Результат: [1, 2, 3]
```

### 4. `Comparator.reverseOrder()`
Метод `reverseOrder()` возвращает компаратор, который сравнивает элементы в **обратном порядке** относительно их естественного порядка.

Пример:
```java
Comparator<Integer> reverseOrderComparator = Comparator.reverseOrder();
List<Integer> numbers = Arrays.asList(3, 1, 2);
numbers.sort(reverseOrderComparator);  // Результат: [3, 2, 1]
```

### 5. `Comparator.nullsFirst()`
Метод `nullsFirst()` возвращает компаратор, который считает `null` меньшим, чем любые другие элементы. Это полезно при работе с коллекциями, где могут присутствовать `null`-значения.

Пример:
```java
Comparator<String> nullsFirstComparator = Comparator.nullsFirst(Comparator.naturalOrder());
List<String> strings = Arrays.asList("apple", null, "banana");
strings.sort(nullsFirstComparator);  // Результат: [null, "apple", "banana"]
```

### 6. `Comparator.nullsLast()`
Метод `nullsLast()` возвращает компаратор, который считает `null` большим, чем любые другие элементы. Это противоположность методу `nullsFirst()`.

Пример:
```java
Comparator<String> nullsLastComparator = Comparator.nullsLast(Comparator.naturalOrder());
List<String> strings = Arrays.asList("apple", null, "banana");
strings.sort(nullsLastComparator);  // Результат: ["apple", "banana", null]
```

### Полный пример использования различных методов `Comparator`:

Предположим, у нас есть класс `Employee`:
```java
public class Employee {
    private String name;
    private int age;
    private double salary;

    // Конструкторы, геттеры и сеттеры

    public String getName() { return name; }
    public int getAge() { return age; }
    public double getSalary() { return salary; }
}
```

Теперь создадим компаратор для сортировки сотрудников по нескольким критериям:
1. По имени.
2. Если имена одинаковы — по возрасту.
3. Если возраст одинаков — по зарплате.
4. С поддержкой `null` для имен (например, `null` считается больше всех).

```java
import java.util.Comparator;

Comparator<Employee> employeeComparator = Comparator
    .comparing(Employee::getName, Comparator.nullsLast(Comparator.naturalOrder()))  // Имя, null-сначала
    .thenComparing(Employee::getAge)  // Возраст
    .thenComparing(Employee::getSalary);  // Зарплата

// Пример использования:
List<Employee> employees = Arrays.asList(
    new Employee("Alice", 30, 5000),
    new Employee("Bob", 25, 7000),
    new Employee("Charlie", 25, 6000),
    new Employee(null, 40, 8000)
);

employees.sort(employeeComparator);
```

Этот пример создаст компаратор, который:
1. Сортирует сотрудников по имени в естественном порядке.
2. Если имена равны, сравнивает по возрасту.
3. Если возраст равен, сравнивает по зарплате.
4. Значение `null` для имени считается последним.

### Итог:
Использование статических методов интерфейса `Comparator` позволяет:
- Легко создавать компараторы для различных полей объектов.
- Поддерживать сложные цепочки сравнений с несколькими уровнями.
- Обрабатывать `null`-значения при сравнении.
- Работать с естественным или обратным порядком сортировки.

Это делает код чище, лаконичнее и понятнее по сравнению с классическими подходами создания компараторов.

