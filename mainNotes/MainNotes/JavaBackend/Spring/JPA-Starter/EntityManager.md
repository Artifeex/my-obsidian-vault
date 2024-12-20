EntityManager - это обертка над Connection. Бин этого класса предоставляется нам автоконфигурацией спринг бута. Причем он является прокси, т.к. создается динамически при старте приложения при оборачивании Connection.

Это интерфейс, который описывает API для всех основных операций над Entity, получение данных и других сущностей JPA. По сути главный API для работы с JPA. Основные операции:
1) Для операций над Entity: persist (добавление Entity под управление JPA), merge (обновление), remove (удаления), refresh (обновление данных), detach (удаление из управление JPA), lock (блокирование Entity от изменений в других thread),
2) Получение данных: find (поиск и получение Entity), createQuery, createNamedQuery, createNativeQuery, contains, createNamedStoredProcedureQuery, createStoredProcedureQuery
3) Получение других сущностей JPA: getTransaction, getEntityManagerFactory, getCriteriaBuilder, getMetamodel, getDelegate
4) Работа с EntityGraph: createEntityGraph, getEntityGraph
5) Общие операции над EntityManager или всеми Entities: close, isOpen, getProperties, setProperty, clear.
Он используется в двух сценариях:
1. [[@Transactional]]
2. Ручного управления транзакциями, с помощью инжекта этого объекта в наш класс. 