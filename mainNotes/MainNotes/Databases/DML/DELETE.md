Используется для удаления данных. 
```SQL
DELETE FROM EMPLOYEE where salary is null;

DELETE FROM EMPLOYEE WHERE SALARY = (SELECT max(salary) from employee);
```
Возвращает количество удаленных строк. Используя RETURNING в конце можно получить сами удаленные строки.