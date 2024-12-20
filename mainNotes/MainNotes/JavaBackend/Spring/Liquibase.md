Для накатывания DDL(т.е. схемы БД) мы раньше использовали Hibenate для этого подхода. Подробнее в [[In memory DB. H2]].
Но в реальных приложениях так не делают, а используют migration framework, которым и является Liquibase.

### Как устроен Liquibase

![[Pasted image 20240909222323.png]]
Есть 2 таблицы. databasechangelogLOCK используется для того, чтобы заблокировать возможность других процессов накатывать свои скрипты параллельно с нами. Т.к. у микросервисов может быть несколько инстансов и они могут все начать ломиться накатывать скрипты на БД.
И вторая таблица databasechangelog. Она нужна для того, чтобы определять какие скрипты уже были использованы, а какие нет. Перед использованием скрипта liquibase проверяет, не был ли он использован, посмотрев на id скрипта и id в таблице databasechangelog. Если в таблице нет такого id, то скрипт не использовался, он используется и в эту таблицу добавляется запись с id скрипта. Если же в таблице уже был такой же id, как у скрипта, который хотим запустить, то скрипт проигнорируется и не будет запущен.

![[Pasted image 20240909223528.png]]
changelog - это наш файл со скриптом. changelog внутри себя хранит changeset(он выполняется в одной транзакции), а каждый changeset может хранить несколько change(уже сами изменения). Хоть best practice по liquibase говорят о том, чтобы мы использовали 1 change для changeset, поскольку тогда будет удобно делать rollback и откатывать только 1 какое-то изменение, а не сразу множество change.

changelog - может быть много(это как раз 1 запись в таблице databasechangelog). Поэтому в liquibase обычно есть один мастер changelog, внутри которого подключаются changelog другие. И раз их много, то их нужно именовать и можно, например, именовать по версиям. 
![[Pasted image 20240909223739.png]]

![[Pasted image 20240909224717.png]]

Т.к. каждый changeset выполняется внутри 1 транзакции, то мы можем определить rollback поведение, которое будет выполняться после выполнения транзакции.
![[Pasted image 20240909224827.png]]

Liquibase реально крутой фреймворк. Нужно будет его потом подробнее посмотреть. 

### Использование
1. Подключаем зависимость liquibase-core
2. В файле application.yml включаем liquibase.
```yml
   spring:  
	  liquibase:  
	    enabled: true
```
3. Написать файл changelog(мастер файл). Содержит список миграций changeset, которые применяются к БД. Миграции выполняются последовательно в том порядке, в котором записаны. Liquibase при старте работы ищет файл db.changelog-master в директории src/main/resources/db/changelog. Т.е. мы должны создать такую диреткорию и файлик db.changelog-master. Мастер файл может быть в разных форматах.
   ![[Pasted image 20241023100709.png]]
   4. Далее в файле db.changelog-master.yml мы будем указывать последовательно changesets, которые хотим выполнить. Для changeset рекомендуется создать директорию. А в файле db.changelog-master.yml мы прописываем путь до каждого changeset. Сам changeset представляет собой одну транзакцию, которая будет выполнена.
   5. author:id - такое пишем в changeset, чтобы задать автора и id. id должно быть уникальным для каждого changeset в рамках одного автора. 

Помимо миграций, создаются еще таблицы для работы liquibase.
- Таблица databasechangelog - в ней хранится информация по примененным changeset-ам. Т.е. с помощью этой таблицы liquibase при запуске понимает, какие из changeset уже были применены и следовательно не нужно их применять. Для этого используется id, который мы задаем при написании changeset.
- Таблица databasechangeloglock - когда у нас несколько реплик, то может быть установлена блокировка одним из приложений, которое начал выполнять миграцию. Т.е. в один момент могут захотеть применять миграции сразу несколько реплик приложения и чтобы не появились проблемы, связанные с этим и используется databasechangeloglock.
  Важное замечание - иногда может получиться так, что locked не изменится на false, т.е. останется true и тогда наше приложение не сможет стартовать из-за этой блокировки. Но не пугаемся, а просто руками исправляем это поле на false.
  ![[Pasted image 20241023103448.png]]
  Также в liquibase есть механизм preconditions с помощью которого мы можем устанавливать условия, при которых будет выполняться changeset. Это, например, удобно, если мы стерли информцию в таблице databasechangelog, то liquibase по новой будет применять changesets, которые мы прописали, т.к. нет информции о том, что они применялись. И если мы в changeset создавали таблицу, то вторую такую мы уже не сможем создать, поскольку она уже существует и вот тут может помочь механизм precondtions.

По итогу создаем такую структуру:
![[Pasted image 20241023114058.png]]
master - это самый главный файл, в него мы будем включать accumulate файлы для каждой версии. Содержимое master:
```yml
databaseChangeLog:  
  - include:  
      relativeToChangelogFile: true  
      file: v1/v1-accumulate-changelog.yml
```
В accumulate-changelog будем включать уже конечные changlogs, внутри которых будут находиться changeset. Содержимое accumulate файла:
```yaml 
databaseChangeLog:  
  - include:  
      relativeToChangelogFile: true  
      file: v1-create-tables-changelog.sql
```
И внутри creaate-tables уже пишем сами changeset:
Содержимое create-tables:
```yml
--liquibase formatted sql  
  
--changeset sandr:1.0  
CREATE TABLE IF NOT EXISTS genre  
(  
    id   INT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,  
    name VARCHAR(128) NOT NULL  
);  
  
CREATE TABLE IF NOT EXISTS book  
(  
    id          INT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,  
    title        VARCHAR(128)              NOT NULL,  
    year        INT CHECK ( year >= 100 AND year <= EXTRACT(YEAR FROM CURRENT_DATE) ),  
    description TEXT  
);
```
--liquibase formatted sql  - обязательная строчка, если миграцию пишем в sql формате.
--changeset sandr:1.0  - author:id. id - должно быть уникально для каждого changeset для одного и того же автора.

А дальше мы просто будем создавать файлик changelog, внутри которого писать changeset, который что-то меняет. А потом добавлять этот файл в accumulate-changelog.

Для применения миграций можно вызвать mvn liquibase:update, либо запустить приложение.
