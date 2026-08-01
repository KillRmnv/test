# Producer
``` java
Properties props = new Properties(); props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092"); props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName()); props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName()); KafkaProducer<String, String> producer = new KafkaProducer<>(props); ProducerRecord<String, String> record = new ProducerRecord<>("orders", "orderId123", "Order created"); producer.send(record); producer.close();
```

## 🔹 `BOOTSTRAP_SERVERS_CONFIG`

`props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");`

### Что это

Адрес(а) Kafka брокеров, к которым клиент **первично** подключится.

### Важно понимать

Это **не список всех брокеров**.

Kafka работает так:

1. Producer подключается к одному брокеру
    
2. Получает от него **metadata кластера** (кто лидер для каких partition)
    
3. Дальше общается уже напрямую с нужными брокерами
    

Поэтому достаточно **1 адреса**, но в проде пишут 2–3 для отказоустойчивости:

## 🔹 `KEY_SERIALIZER_CLASS_CONFIG`

`props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,           StringSerializer.class.getName());`

### Что такое Serializer вообще?

Kafka **работает только с байтами**.

Она не знает, что такое String, Object, JSON, Avro.

Поэтому Producer обязан уметь:

> превратить key из Java-объекта → в `byte[]`

Этим занимается **Serializer**.

### Почему key важен

Key используется для:

- выбора partition
    
- гарантии порядка сообщений
    

Если key одинаковый → попадёт в одну partition.

## `VALUE_SERIALIZER_CLASS_CONFIG`

`props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,           StringSerializer.class.getName());`

То же самое, но для тела сообщения (value).

На практике тут часто:

- `StringSerializer`
    
- `JsonSerializer`
    
- `ByteArraySerializer`
    
- Avro/Protobuf сериализаторы
  
  
  ##  ProducerRecord

`ProducerRecord<String, String> record =         new ProducerRecord<>("orders", "orderId123", "Order created");`

Это объект одного сообщения.

Конструктор:

`ProducerRecord(topic, key, value)`

### Что внутри record:

|Поле|Значение|Для чего|
|---|---|---|
|topic|`"orders"`|Куда отправить|
|key|`"orderId123"`|Влияет на partition|
|value|`"Order created"`|Тело сообщения|
|partition|null|Kafka сама выберет|
|timestamp|auto|Время отправки|
## `producer.send(record);`

Самое интересное место.

Это **НЕ** отправка в сеть сразу.

Что происходит:

1. Сообщение кладётся во внутренний буфер
    
2. Kafka ждёт, пока накопится batch (по размеру или времени)
    
3. Отправляет пачкой
    

Это делает Kafka очень быстрой.

### По умолчанию `send` — асинхронный

Метод возвращает `Future<RecordMetadata>`.

Если ты просто вызываешь `send()` и выходишь из программы — сообщение может не успеть уйти.

## `producer.close();`

Очень важная строка.

`close()`:

- форсит отправку всех батчей
    
- закрывает соединения
    

Если её убрать — сообщения могут потеряться.

В проде чаще делают:

`producer.flush(); producer.close();`


## Consumer

## `Properties props = new Properties();`

Та же конфигурационная мапа.  
Но у consumer настроек **в разы больше и они критичнее**, чем у producer.

## `BOOTSTRAP_SERVERS_CONFIG`

`props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");`

То же самое, что у producer:  
точка входа, чтобы получить **metadata** о кластере и партициях.

## `GROUP_ID_CONFIG` — самое важное место во всём consumer

`props.put(ConsumerConfig.GROUP_ID_CONFIG, "order-processors");`

Это **идентификатор consumer group**.

Kafka работает так:

> Не «кто читает топик», а **какая группа читает топик**.

### Что это даёт

Если запустить 3 одинаковых приложения с этим `group.id`:

- Kafka разобьёт partitions между ними
    
- Они будут читать параллельно
    
- Одно и то же сообщение **НЕ** попадёт двум consumer'ам
    

Если поменять `group.id` → Kafka считает, что это **новый читатель**, и даст читать всё заново.

👉 Это частый вопрос: _«Как перечитать весь топик?»_  
Ответ: сменить group.id

## `AUTO_OFFSET_RESET_CONFIG`

`props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");`

Очень важная штука, которую почти никто не понимает правильно.

Работает **ТОЛЬКО когда для этой группы ещё нет offset**.

### Варианты:

|Значение|Что значит|
|---|---|
|earliest|читать с самого начала топика|
|latest|читать только новые сообщения|
|none|упасть с ошибкой|

То есть:

- первый запуск группы → сработает
    
- дальше Kafka уже помнит offset → эта настройка игнорируется
  
  
  ## `KafkaConsumer<String, String> consumer`

Создание клиента.

Внутри:

- TCP соединения
    
- heartbeat к Kafka (чтобы группа знала, что ты жив)
    
- управление offsets
    
- ребаланс группы
    

---

## 🔥 `consumer.subscribe(List.of("orders"));`

Ты **не говоришь**, какие partitions читать.  
Ты говоришь: _«Я хочу читать этот топик в составе группы»_.

Kafka сама решает:

- какие partitions тебе дать
    
- когда их отобрать (rebalance)
  
  
  ## `poll(Duration.ofMillis(100))` — самая важная строка

`ConsumerRecords<String, String> records =         consumer.poll(Duration.ofMillis(100));`

### Что делает `poll` на самом деле:

1. Забирает сообщения с брокера
    
2. Отправляет heartbeat (я жив)
    
3. Проверяет, не случился ли rebalance
    
4. Обновляет offsets
    

Если **долго не вызывать poll** → Kafka считает consumer мёртвым → rebalance.

Вот откуда на проде «почему нас постоянно ребалансит???»