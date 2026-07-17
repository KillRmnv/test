### 1. JUnit vs TestNG: В чем разница и что учить?

| Характеристика | **JUnit 5 (Jupiter)** | **TestNG** |
| :--- | :--- | :--- |
| **Стиль** | Современный, более простой, «идиоматичный» для Java 8+. | Более мощный, с большим количеством встроенных фич. |
| **Аннотации** | `@Test`, `@BeforeEach`, `@AfterEach` | `@Test`, `@BeforeMethod`, `@AfterMethod` |
| **Параметризация** | `@ParameterizedTest` + `@CsvSource` / `@MethodSource` | `@DataProvider` (более гибкий) |
| **Параллельный запуск** | Есть, но сложнее настроить. | Отличная встроенная поддержка. |
| **Зависимости между тестами** | Не рекомендуется (тесты должны быть изолированы). | Есть (`dependsOnMethods`), но злоупотреблять не стоит. |
| **Где используют в EPAM?** | Чаще всего в новых проектах и микросервисах. | Часто в легаси, крупных enterprise-проектах, где нужна гибкая конфигурация. |

**Ваш ответ на собеседовании:** *«Я хорошо знаком с JUnit 5, потому что это стандарт для современных Spring Boot-приложений. Но я понимаю, что в некоторых проектах EPAM используется TestNG, и я готов быстро переключиться, так как их синтаксис очень похож, а концепции жизненного цикла идентичны.»*

---

### 2. Аннотации жизненного цикла теста (JUnit 5)

Это спросят 100%. Вас могут попросить объяснить, в каком порядке выполняются методы.

**Пример кода:**

```java
import org.junit.jupiter.api.*;

public class LifecycleTest {

    @BeforeAll
    static void initAll() {
        System.out.println("1. Выполняется 1 раз перед ВСЕМИ тестами в классе (статический метод)");
        // Например: поднять тяжелый контекст Spring или открыть соединение с БД
    }

    @BeforeEach
    void init() {
        System.out.println("2. Выполняется ПЕРЕД каждым тестом");
        // Например: создать новый объект User, очистить кэш
    }

    @Test
    void testOne() {
        System.out.println("3. Тест #1");
    }

    @Test
    void testTwo() {
        System.out.println("3. Тест #2");
    }

    @AfterEach
    void tearDown() {
        System.out.println("4. Выполняется ПОСЛЕ каждого теста");
        // Например: удалить созданные данные из БД, закрыть WebDriver
    }

    @AfterAll
    static void tearDownAll() {
        System.out.println("5. Выполняется 1 раз после ВСЕХ тестов (статический метод)");
        // Например: закрыть соединение с БД, закрыть пул потоков
    }
}
```

**Реальный порядок вывода в консоль для 2 тестов:**
```
1. Выполняется 1 раз перед ВСЕМИ тестами в классе
2. Выполняется ПЕРЕД каждым тестом
3. Тест #1
4. Выполняется ПОСЛЕ каждого теста
5. Выполняется ПЕРЕД каждым тестом
6. Тест #2
7. Выполняется ПОСЛЕ каждого теста
8. Выполняется 1 раз после ВСЕХ тестов
```

**Для TestNG это будет:**
- `@BeforeSuite` / `@AfterSuite` (уровень всего набора тестов)
- `@BeforeTest` / `@AfterTest` (уровень `<test>`-тега в testng.xml)
- `@BeforeClass` / `@AfterClass` (аналог JUnit `@BeforeAll` / `@AfterAll`)
- `@BeforeMethod` / `@AfterMethod` (аналог JUnit `@BeforeEach` / `@AfterEach`)

---

### 3. Ассерты (Assertions) — проверка ожидаемого результата

Это сердце любого теста. Вы должны уметь написать проверку на лету.

**JUnit 5 (Jupiter):**

```java
import static org.junit.jupiter.api.Assertions.*;

@Test
void testAssertions() {
    // 1. Проверка на равенство
    assertEquals(5, 2 + 3, "Сообщение, если тест упадет");

    // 2. Проверка на true / false
    assertTrue(10 > 5);
    assertFalse(10 < 5);

    // 3. Проверка на null / not null
    String str = null;
    assertNull(str);
    assertNotNull(new Object());

    // 4. Проверка на выброс исключения (важно для API-тестов!)
    assertThrows(IllegalArgumentException.class, () -> {
        Integer.parseInt("abc"); // Это выбросит NumberFormatException (наследник IllegalArgumentException)
    });

    // 5. Групповые проверки (JUnit 5)
    assertAll(
        () -> assertEquals(5, 2 + 3),
        () -> assertTrue("Hello".contains("ell"))
    );
}
```

**TestNG:**

```java
import org.testng.Assert;

@Test
void testAssertions() {
    Assert.assertEquals(5, 2 + 3);
    Assert.assertTrue(10 > 5);
    Assert.assertNull(null);
    Assert.assertThrows(NumberFormatException.class, () -> Integer.parseInt("abc"));
}
```

---

### 4. Параметризованные тесты — ваша суперсила

Параметризованные тесты позволяют запустить **один и тот же тест** с **разными наборами данных**. Это идеально подходит для проверки **граничных значений** (BVA), которые мы с вами разбирали ранее.

#### Вариант А: `@CsvSource` (самый простой и частый на собесе)

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

@ParameterizedTest
@CsvSource({
    "18, true",   // 18 — валидный возраст
    "17, false",  // 17 — невалидный (граница)
    "60, true",   // 60 — валидный (граница)
    "61, false"   // 61 — невалидный
})
void testAgeValidation(int age, boolean expectedResult) {
    boolean isValid = ageValidator.isValid(age);
    assertEquals(expectedResult, isValid);
}
```

#### Вариант Б: `@MethodSource` (для сложных объектов)

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;

@ParameterizedTest
@MethodSource("provideUsers")
void testUserCreation(User user, boolean expectedResult) {
    boolean result = userService.createUser(user);
    assertEquals(expectedResult, result);
}

static Stream<Arguments> provideUsers() {
    return Stream.of(
        Arguments.of(new User("john@mail.com", 25), true),
        Arguments.of(new User(null, 30), false),
        Arguments.of(new User("jane@mail.com", 17), false)
    );
}
```

#### Для TestNG: `@DataProvider`

```java
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

@Test(dataProvider = "ageData")
void testAgeValidation(int age, boolean expectedResult) {
    boolean isValid = ageValidator.isValid(age);
    Assert.assertEquals(isValid, expectedResult);
}

@DataProvider(name = "ageData")
public Object[][] provideData() {
    return new Object[][] {
        {18, true},
        {17, false},
        {60, true},
        {61, false}
    };
}
```

---

### 5. Советы для собеседования в EPAM

1.  **Говорите о TDD:** Если спросят *«Зачем нужны аннотации?»*, ответьте: *«Они позволяют организовать **чистую структуру тестов**. Например, `@BeforeEach` гарантирует, что каждый тест стартует с чистого состояния (позволяет избежать Flaky Tests), что соответствует принципам TDD (Test-Driven Development) и делает тесты надежными.»*

2.  **Свяжите это с реальным проектом:** *«В своих учебных проектах я писал параметризованные тесты для проверки валидации входных данных. Например, вместо 10 одинаковых тестов на разные email-адреса я использовал `@CsvSource`, что сократило код в 3 раза.»*

3.  **Упомяните про @Tag (группировка):**
    *   `@Tag("smoke")` — тесты, которые бегут на каждую сборку.
    *   `@Tag("regression")` — тесты, которые бегут перед релизом.
    *   Это важно для настройки CI/CD.

---

### 6. Резюме (Шпаргалка)

| Концепция | JUnit 5 | TestNG |
| :--- | :--- | :--- |
| **Перед всеми тестами** | `@BeforeAll static` | `@BeforeClass` |
| **Перед каждым тестом** | `@BeforeEach` | `@BeforeMethod` |
| **После каждого теста** | `@AfterEach` | `@AfterMethod` |
| **После всех тестов** | `@AfterAll static` | `@AfterClass` |
| **Равенство** | `assertEquals(a, b)` | `Assert.assertEquals(a, b)` |
| **True/False** | `assertTrue(cond)` | `Assert.assertTrue(cond)` |
| **Исключение** | `assertThrows(Ex.class, () -> {...})` | `Assert.assertThrows(Ex.class, () -> {...})` |
| **Параметризация** | `@ParameterizedTest` + `@CsvSource` / `@MethodSource` | `@Test(dataProvider = "name")` + `@DataProvider` |

---

**Что дальше?** 
Мы разобрали JUnit и TestNG. Остались еще **критические блоки**, которые точно спросят:
- **Selenium WebDriver:** Локаторы, ожидания, Page Object Model (ваша super-skill!).
- **Postman / REST API:** 100% спросят, так как это база любого QA.
- **SQL:** Базовые запросы (SELECT, JOIN, GROUP BY) — дают прямо на собеседовании писать.

Какой блок разбираем следующим?