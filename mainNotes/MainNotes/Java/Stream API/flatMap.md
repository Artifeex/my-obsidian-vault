На вход принимает метод, который должен создать новый стрим. И потом все эти стримы соединяются в один стрим. Т.е. до flatMap, у нас мог быть, например, Stream< String>, дальше внутри flatMap мы передали метод, который принимает String, а возвращает Stream< String>. И потом все эти стримы объединяются и после flatMap мы имеем все также Stream< String>.
### Пример использования:

#### 1. **С `List<String>` и разбиением строки на слова**:

Предположим, у нас есть список строк, и каждая строка представляет предложение. Мы хотим преобразовать эти предложения в поток слов.

##### Пример без `flatMap()`:

```java
List<String> sentences = Arrays.asList("Hello world", "Java streams are powerful");  
List<Stream<String>> wordStreams = sentences.stream()
		.map(sentence -> Arrays.stream(sentence.split(" "))).collect(Collectors.toList());`
```

В этом примере каждый элемент списка преобразуется в поток слов, но результатом будет список потоков (`List<Stream<String>>`), что не очень удобно для дальнейшей обработки.

##### Пример с `flatMap()`:

```java
List<String> sentences = Arrays.asList("Hello world", "Java streams are powerful");  
List<String> words = sentences.stream()
.flatMap(sentence -> Arrays.stream(sentence.split(" ")))         .collect(Collectors.toList());
```

Теперь, с использованием `flatMap()`, все потоки слов были "свёрнуты" в один плоский поток (`Stream<String>`), и результатом будет список слов.

### Этапы работы `flatMap()`:

1. Применяется преобразование к каждому элементу потока (например, разделение строки на слова).
2. Каждый элемент превращается в новый поток (в нашем случае — поток слов).
3. Все созданные потоки "сворачиваются" в один общий поток.
4. Этот плоский поток можно далее обрабатывать, фильтровать, сортировать и т.д.

### Разница между `map()` и `flatMap()`:

- **`map()`**: Преобразует каждый элемент потока в новый элемент. Результат — поток того же уровня вложенности.
    - Например: `Stream<String>` -> `Stream<Integer>`
- **`flatMap()`**: Преобразует каждый элемент потока в поток, а затем "сворачивает" все вложенные потоки в один плоский поток.
    - Например: `Stream<List<String>>` -> `Stream<String>`

### Ключевые моменты:

- `flatMap()` используется для работы с вложенными структурами (например, коллекции коллекций).
- Он "разворачивает" несколько потоков в один плоский поток.
- Это удобно, когда нужно объединить данные из нескольких источников в один поток для дальнейшей обработки.