`protected Object clone(Object object) throws CloneNotSupportedException`

Это указывает на то, что хоть метод и есть в классе Object и разработчик желает им воспользоваться, то его нужно переопределить. Для этого нужно реализовать интерфейс Cloneable, чтобы соблюсти контракт иначе выброситься исключение.

### Создает поверхностную копию
Метод `clone()` создает "поверхностную" копию объекта. Это означает, что копируются только примитивные типы данных и ссылки на объекты, но не сами объекты, на которые ссылаются поля. Если объект содержит ссылочные типы, то в новой копии будут скопированы только ссылки на те же объекты, а не их содержимое. Это может привести к тому, что изменения в одном объекте могут затронуть данные в другом.

### Если мы планируем вызывать метод super.clone(), то необходимо реализовать интерфейс Cloneable - интерфейс маркер. Метод clone класса Object проверяет перед клонированием реализует ли класс такой интерфейс

### Т.к. данный метод protected, то в классах, в которых мы хотим это использовать нужно переопределить этот метод с модификатором public

### Пример реализации глубокого клонирования
```java
class MyClass implements Cloneable {
	int value; 
	int[] arr; 
	MyClass(int value, int[] arr) { 
		this.value = value; this.arr = arr; 
	} 
	
	@Override public Object clone() throws CloneNotSupportedException { 
		//глубокое копирование массива
		MyClass cloned = (MyClass) super.clone(); cloned.arr = arr.clone(); 
		return cloned; 
	} 
}
```


Для клонирования объекта в Java можно воспользоваться тремя способами:
1. Переопределение метода clone() и реализация интерфейса Cloneable().
    
2. Использование конструктора копирования.
    
3. Использовать для клонирования механизм сериализации.
    