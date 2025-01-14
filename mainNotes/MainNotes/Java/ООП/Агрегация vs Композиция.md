Да, есть разница между **композицией** и **агрегацией** в объектно-ориентированном программировании (ООП). Эти понятия описывают отношения **"часть-целое"** между объектами, но различаются по уровню зависимости и жизненного цикла компонентов.

---

### Композиция

Композиция — это отношение **жесткой зависимости** между объектами. Подразумевается, что:

- **Часть не может существовать без целого**.
- Если уничтожается объект-целое, уничтожаются и все его части.
- Части управляются объектом-целым.

#### Пример:

Представьте, что объект **"Дом"** состоит из объектов **"Комната"**. Комнаты не могут существовать без дома.

```java
class Room {
    private String name;
    public Room(String name) {
        this.name = name;
    }
}

class House {
    private List<Room> rooms = new ArrayList<>();

    public House() {
        rooms.add(new Room("Living Room"));
        rooms.add(new Room("Bedroom"));
    }
}
```

- Здесь `Room` является частью `House`. Без объекта `House` комнаты теряют смысл.

---

### Агрегация

Агрегация — это отношение **слабой зависимости** между объектами. Подразумевается, что:

- **Часть может существовать независимо от целого**.
- Жизненные циклы объектов не связаны.
- Объекты взаимодействуют, но не зависят друг от друга напрямую.

#### Пример:

Представьте, что объект **"Университет"** связан с объектами **"Студент"**. Студенты могут существовать вне университета.

```java
class Student {
    private String name;
    public Student(String name) {
        this.name = name;
    }
}

class University {
    private List<Student> students = new ArrayList<>();

    public void addStudent(Student student) {
        students.add(student);
    }
}
```

- Здесь `Student` может существовать независимо от `University`. Университет только агрегирует студентов.

---

### Ключевые отличия

|**Характеристика**|**Композиция**|**Агрегация**|
|---|---|---|
|**Зависимость**|Жесткая (часть не может существовать без целого).|Слабая (часть может существовать отдельно).|
|**Жизненный цикл**|Связан: уничтожение целого приводит к уничтожению частей.|Независимый: уничтожение целого не влияет на части.|
|**Взаимодействие**|Часть является неотъемлемой частью целого.|Часть может быть частью нескольких объектов.|
|**Пример**|Дом и Комнаты.|Университет и Студенты.|

---

### Диаграммы UML

- **Композиция:** Отображается черным ромбом у объекта-целого.
- **Агрегация:** Отображается белым ромбом у объекта-целого.

---

### Когда использовать?

- **Композиция**:
    
    - Если целое управляет частями, и они не имеют смысла без него.
    - Например, класс `Car` и его компоненты (`Engine`, `Wheel`).
- **Агрегация**:
    
    - Если объекты могут существовать независимо друг от друга.
    - Например, `Library` и `Book`. Книга может быть взята из библиотеки и существовать отдельно.