Используется для замены значения, которое могло быть NULL.
```SQL
SELECT COALESCE(u.name, "defaultName")
FROM users u;
```