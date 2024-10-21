Базовый класс от которого неявно наследуются все классы в Java.

### Методы класса Object
1. `public final native Class getClass()` — возвращает в рантайме класс данного объекта.
    
2. `public native int hashCode()` — возвращает хеш-код
    
3. `public boolean equals(Object obj)` — сравнивает объекты.
    
4. `protected native Object clone() throws CloneNotSupportedException` — клонирование объекта
    
5. `public String toString()` — возвращает строковое представление объекта.
    
6. `public final native void notify()` — просыпается один поток, который ждет на “мониторе” данного объекта.
    
7. `public final native void notifyAll()` — просыпаются все потоки, которые ждут на “мониторе” данного объекта.
    
8. `public final native void wait(long timeout) throws InterruptedException` — поток переходит в режим ожидания в течение указанного времени.
    
9. `public final void wait() throws InterruptedException` — приводит данный поток в ожидание, пока другой поток не вызовет notify() или notifyAll() методы для этого объекта.
    
10. `public final void wait(long timeout, int nanos) throws InterruptedException` — приводит данный поток в ожидание, пока другой поток не вызовет notify() или notifyAll() для этого метода, или пока не истечет указанный промежуток времени.
    
11.  `protected void finalize() throws Throwable` — вызывается сборщиком мусора, когда garbage collector определил, что ссылок на объект больше нет.
    

******