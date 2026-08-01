# ArgumentCaptor и verify в Mockito

## verify() - проверка вызовов
```java
// Проверка, что метод вызван 1 раз
verify(mockObject).methodName();

// С указанием количества
verify(mockObject, times(3)).methodName();    // ровно 3
verify(mockObject, never()).methodName();     // не вызван
verify(mockObject, atLeastOnce()).methodName(); // ≥1
verify(mockObject, atLeast(2)).methodName();  // ≥2
verify(mockObject, atMost(5)).methodName();   // ≤5
```

## ArgumentCaptor - захват аргументов
```java
// 1. Создать захватчик
ArgumentCaptor<Event> captor = ArgumentCaptor.forClass(Event.class);

// 2. Захватить аргумент при вызове
verify(eventRepository).save(captor.capture());

// 3. Получить захваченное значение
Event actual = captor.getValue();

// 4. Проверить свойства
assertThat(actual.getTitle()).isEqualTo("Rock Concert");

// Для нескольких вызовов
List<Event> allEvents = captor.getAllValues();
```

## Зачем нужен ArgumentCaptor?
- **Без него**: проверяем только факт вызова `verify(repo).save(any(Event.class))`
- **С ним**: проверяем, ЧТО именно передали в метод

## Важные нюансы
```java
// ✅ Правильный порядок
verify(repo).save(captor.capture());
Event actual = captor.getValue();

// ❌ Ошибка - getValue() до verify()
Event actual = captor.getValue(); // Ничего не захвачено!
```

## Альтернатива - matcher
```java
verify(eventRepository).save(argThat(event -> 
    event.getTitle().equals("Rock Concert") &&
    event.getVenue().getId() == 10
));
```