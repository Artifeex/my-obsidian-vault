Да, вы правы: **HTTP** по умолчанию работает поверх **TCP** (Transmission Control Protocol). Однако есть несколько важных деталей, связанных с тем, как HTTP управляет соединениями и обрабатывает запросы и ответы.

### Основные аспекты работы HTTP и TCP

1. **Использование TCP**:
   - HTTP является приложенческой протоколом, который строится на основе транспортного протокола TCP. TCP обеспечивает надежный и последовательный обмен данными, гарантируя, что все пакеты данных доставляются без потерь и в правильном порядке.

2. **Открытие и закрытие соединений**:
   - В классическом HTTP/1.0 поведение действительно предполагало разрыв соединения после получения ответа. Это означает, что для каждого запроса создается новое TCP-соединение, которое открывается, используется для передачи данных и затем закрывается. Это может быть неэффективно, особенно если клиент отправляет несколько запросов на один и тот же сервер.

3. **HTTP/1.1 и постоянные соединения**:
   - В HTTP/1.1 было введено новое поведение, позволяющее поддерживать **постоянные соединения** (или **keep-alive**). Это позволяет серверу и клиенту поддерживать одно TCP-соединение открытым для обработки нескольких запросов и ответов. Таким образом, после завершения одного запроса и ответа соединение может оставаться открытым для других запросов. Это значительно снижает накладные расходы, связанные с открытием и закрытием соединений.

4. **Тайм-ауты**:
   - Несмотря на возможность использования постоянных соединений, HTTP/1.1 все же поддерживает тайм-ауты. Если соединение остается неактивным в течение определенного периода времени, оно может быть автоматически закрыто. Это позволяет избежать излишнего использования ресурсов на сервере и клиенте.

5. **HTTP/2**:
   - В HTTP/2, который является более новым стандартом, была добавлена еще большая оптимизация работы с соединениями. HTTP/2 использует **мультиплексирование**, что позволяет отправлять несколько запросов и получать ответы одновременно по одному TCP-соединению. Это улучшает производительность и уменьшает задержку.

### Резюме

- **HTTP по умолчанию работает поверх TCP** и использует его для надежной передачи данных.
- В **HTTP/1.0** соединение разрывается после получения ответа, тогда как **HTTP/1.1** позволяет поддерживать постоянные соединения, что улучшает производительность.
- **HTTP/2** оптимизирует соединения еще больше с помощью мультиплексирования.

Это делает HTTP более эффективным и быстрым в обмене данными между клиентом и сервером.

[[HTTP-Stateless]]
[[HTTPS]]