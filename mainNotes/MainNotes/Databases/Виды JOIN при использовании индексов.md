
- Nested Loop - срабатывает, когда мало данных(например, LIMIT 100). Считывает одну таблицу полностью, а для второй использует индексы и связывает.
- Hash Join - считывает полностью и первую и вторую таблицу, но по одной из них строит HashMap и потом связывает.
- Merge Join - работает только на основании отсортированных ключей. Связка происходит как раз за счет того, что в обоих таблицах ключи отсортированы и можно идти по ключам и сравнивать их друг за другом.