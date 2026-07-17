**Assertions (утверждения / проверки)** — это логическое ядро любого теста. Без них тест не имеет смысла, так как именно ассерты сравнивают фактический результат работы программы с ожидаемым. Если проверка не проходит, фреймворк выбрасывает исключение (например, `AssertionError`), тест прерывается и помечается красным.

В JUnit 5 и TestNG работа с ассертами устроена похоже, но есть критические архитектурные различия в синтаксисе параметров и поддержке мягких проверок (Soft Assertions).

## 1. Базовые типы проверок

В обоих фреймворках есть стандартный набор методов для базовых проверок. Все они принимают необязательный параметр — **текстовое сообщение**, которое выведется в консоль или отчет, если тест упадет.

|**Метод**|**Что проверяет**|
|---|---|
|`assertEquals(expected, actual)`|Равенство двух значений (объектов, чисел, строк).|
|`assertNotEquals(unexpected, actual)`|Проверяет, что фактическое значение _не равно_ указанному.|
|`assertTrue(condition)`|Проверяет, что условие истинно (`true`).|
|`assertFalse(condition)`|Проверяет, что условие ложно (`false`).|
|`assertNull(object)`|Проверяет, что ссылка на объект равна `null`.|
|`assertNotNull(object)`|Проверяет, что объект существует (не равен `null`).|
|`assertSame(obj1, obj2)`|Проверяет, что обе переменные ссылаются на **один и тот же** объект в памяти (`==`).|
|`assertNotSame(obj1, obj2)`|Проверяет, что ссылки ведут на разные объекты.|

## 2. Разница в синтаксисе (JUnit 5 vs TestNG)

Это одна из самых частых «ловушек» на собеседованиях. Разработчики этих библиотек по-разному расположили аргументы в сигнатуре методов.

- **В JUnit 5** порядок такой: `(Expected, Actual, Message)`. Сообщение идет в самом конце (часто как лямбда-выражение для ленивой инициализации).
    
- **В TestNG** порядок другой: `(Actual, Expected, Message)`. Сначала идет фактическое значение, потом ожидаемое, а сообщение — в конце.
    

Java

```
// --- JUnit 5 ---
import static org.junit.jupiter.api.Assertions.assertEquals;
// JUnit: Expected -> Actual -> Message
assertEquals(200, response.getStatusCode(), "Код ответа должен быть 200");

// --- TestNG ---
import org.testng.Assert;
// TestNG: Actual -> Expected -> Message
Assert.assertEquals(response.getStatusCode(), 200, "Код ответа должен быть 200");
```

_Если их перепутать в коде, тесты все равно запустятся, но в случае падения отчет покажет неразбериху: Actual и Expected поменяются местами._

## 3. Hard Assertions vs. Soft Assertions

Это фундаментальное различие, которое напрямую влияет на логику тестирования сложных сценариев.

### Hard Assertions (Жесткие проверки) — используются по умолчанию

Как только жесткий ассерт падает, выполнение текущего `@Test` метода **мгновенно прекращается**. Все последующие строчки кода в этом методе игнорируются.

- _Плюс:_ Быстро падает, экономит время, если дальнейшие проверки не имеют смысла (например, если API вернул 500, то проверять JSON-тело ответа бессмысленно).
    
- _Минус:_ Если у вас на форме 10 полей и упала проверка первого поля, вы не узнаете, работают ли остальные 9, пока не исправите первое.
    

### Soft Assertions (Мягкие проверки)

Позволяют выполнить **все** проверки в тесте, даже если некоторые из них упали. Ошибки накапливаются, а тест падает только в самом конце, выводя список всех несоответствий.

#### Реализация в TestNG (Встроено из коробки):

В TestNG для этого есть класс `SoftAssert`. В конце теста **обязательно** нужно вызвать метод `assertAll()`, иначе тест всегда будет проходить как успешный, даже если все проверки внутри упали.

Java

```
import org.testng.annotations.Test;
import org.testng.asserts.SoftAssert;

public class ProfileTest {
    @Test
    public void testUserProfile() {
        SoftAssert softAssert = new SoftAssert();
        
        softAssert.assertEquals(user.getName(), "Kirill", "Имя не совпадает");
        softAssert.assertEquals(user.getRole(), "QA", "Роль не совпадает");
        softAssert.assertTrue(user.isActive(), "Пользователь не активен");
        
        // КРИТИЧЕСКИ ВАЖНО: собирает все упавшие проверки и роняет тест, если были ошибки
        softAssert.assertAll(); 
    }
}
```

#### Реализация в JUnit 5:

В JUnit 5 нет класса `SoftAssert`, но есть более элегантное решение — `assertAll()`, которое принимает на вход группу исполняемых лямбда-выражений (executable).

Java

```
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

public class ProfileTest {
    @Test
    void testUserProfile() {
        // Выполнятся абсолютно все проверки внутри блока. 
        // Если упадут две из трех, JUnit покажет детальный отчет по обеим ошибкам.
        assertAll("Характеристики пользователя",
            () -> assertEquals("Kirill", user.getName(), "Имя не совпадает"),
            () -> assertEquals("QA", user.getRole(), "Роль не совпадает"),
            () -> assertTrue(user.isActive(), "Пользователь не активен")
        );
    }
}
```

## 4. Проверка исключений (Exceptions)

Иногда ожидаемое поведение программы — это выброс ошибки (например, при передаче невалидных данных в метод).

- **В JUnit 5:** Используется встроенный метод `assertThrows`.
    
    Java
    
    ```
    // Проверяем, что деление на ноль выбрасывает ArithmeticException
    Throwable exception = assertThrows(ArithmeticException.class, () -> {
        calculator.divide(10, 0);
    });
    assertEquals("/ by zero", exception.getMessage());
    ```
    
- **В TestNG:** Исключения проверяются прямо в аннотации `@Test`.
    
    Java
    
    ```
    @Test(expectedExceptions = ArithmeticException.class)
    public void testDivisionByZero() {
        calculator.divide(10, 0);
    }
    ```
    

### 💡 Как ответить на интервью:

> _«Главное различие в базовых ассертах между JUnit и TestNG — это порядок аргументов: JUnit принимает `(expected, actual)`, а TestNG — `(actual, expected)`. Также они по-разному работают с мягкими проверками. В TestNG мы используем класс `SoftAssert` и обязательно вызываем метод `assertAll()` в конце. В JUnit 5 мягкие проверки реализованы через метод `assertAll()`, который группирует лямбда-выражения — это лаконичнее, так как исключает риск забыть вызвать финальный метод сбора ошибок»_.