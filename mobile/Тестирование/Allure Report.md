# Allure Report — что нужно знать QA (особенно с Java-стеком)

## Что такое Allure

**Allure Report** — фреймворк для генерации красивых, детализированных отчётов о результатах тестирования. В отличие от стандартных JUnit XML-отчётов (сухих и малоинформативных), Allure даёт наглядную визуализацию: графики, историю запусков, шаги теста, вложения (скриншоты, логи), группировку по фичам/severity.

Важно понимать разницу на собеседовании: **Allure — это не test runner**, это библиотека для сбора данных о тестах + отдельный инструмент для генерации HTML-отчёта из этих данных.

## Как это работает — механика

```
1. Тесты выполняются (JUnit/TestNG/Selenium)
2. Allure-адаптер (allure-junit5 / allure-testng) собирает данные во время выполнения
   → пишет JSON-файлы в папку allure-results/
3. Allure CLI/Maven-плагин читает allure-results/
   → генерирует HTML-отчёт (allure-report/)
```

## Интеграция в Java/Maven проект (актуально для твоего стека)

### Зависимость в `pom.xml`

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-junit5</artifactId>
    <version>2.29.0</version>
</dependency>
```

### Aspectj для перехвата аннотаций (`@Step`, `@Attachment`)

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <argLine>
            -javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/1.9.22/aspectjweaver-1.9.22.jar"
        </argLine>
    </configuration>
</plugin>
```

## Ключевые аннотации (частый вопрос на собеседовании)

```java
@Epic("Authorization")           // самая крупная группировка (модуль)
@Feature("Login")                // фича внутри Epic
@Story("Login with valid credentials")  // конкретный сценарий
@Severity(SeverityLevel.CRITICAL)       // важность теста
@Description("Verify user can log in with valid email and password")
class LoginTest {

    @Test
    @DisplayName("Successful login test")
    void loginWithValidCredentials() {
        step1_openLoginPage();
        step2_enterCredentials();
        step3_verifyDashboard();
    }

    @Step("Open login page")
    void step1_openLoginPage() {
        driver.get("https://example.com/login");
    }

    @Step("Enter credentials: {email}")
    void step2_enterCredentials(String email) {
        // ...
    }

    @Attachment(value = "Screenshot", type = "image/png")
    byte[] takeScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }
}
```

### Разбор аннотаций

|Аннотация|Назначение|
|---|---|
|`@Epic` / `@Feature` / `@Story`|Иерархическая группировка тестов в отчёте (Epic → Feature → Story)|
|`@Severity`|Критичность: BLOCKER, CRITICAL, NORMAL, MINOR, TRIVIAL — помогает приоритизировать баги при падении тестов|
|`@Step`|Разбивает тест на читаемые шаги в отчёте (видно, на каком именно шаге упал тест)|
|`@Attachment`|Прикрепляет файл к отчёту (скриншот, лог, HAR-файл — вспомним прошлую тему, кстати) при падении теста|
|`@DisplayName`|Человекочитаемое имя теста вместо `loginWithValidCredentials`|
|`@Description`|Текстовое описание, что именно проверяет тест|
|`@Link` / `@Issue` / `@TmsLink`|Ссылки на Jira-тикет, TestRail/TMS кейс — трассируемость требований|

## Автоматическое прикрепление скриншота при падении (частый практический кейс)

```java
@AfterEach
void tearDown(TestInfo testInfo) {
    if (/* тест упал */) {
        Allure.addAttachment("Screenshot on failure", 
            new ByteArrayInputStream(
                ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES)
            ));
    }
    driver.quit();
}
```

Это одна из ключевых практических ценностей Allure — когда тест падает в CI, ты сразу видишь скриншот момента падения, не нужно перезапускать локально, чтобы понять, что произошло.

## Генерация отчёта — команды

```bash
# Локально
mvn clean test
allure serve allure-results   # поднимает локальный сервер и открывает отчёт в браузере

# В CI (без serve, статичная генерация)
allure generate allure-results --clean -o allure-report
```

## Интеграция с CI/CD (связка с прошлой темой)

### GitLab CI пример

```yaml
test:
  stage: test
  script:
    - mvn test
  artifacts:
    paths:
      - allure-results/
    expire_in: 7 days

generate-report:
  stage: report
  script:
    - allure generate allure-results --clean -o allure-report
  artifacts:
    paths:
      - allure-report/
```

### Jenkins — есть готовый плагин **Allure Jenkins Plugin**

```groovy
post {
    always {
        allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
    }
}
```

Плагин сам публикует отчёт прямо в Jenkins UI как отдельную вкладку у job'а — не нужно вручную генерировать HTML.

## Что показывает готовый отчёт (важно для собеседования — уметь описать)

- **Overview** — общая статистика: passed/failed/broken/skipped, графики по времени выполнения
- **Categories** — группировка падений по типу (например, "Product defects" vs "Test defects" — отдельно баги приложения и отдельно проблемы самого теста, типа NoSuchElementException из-за смены верстки)
- **Suites** — тесты по классам/группам
- **Graphs** — Severity breakdown, Duration trend, Retries trend
- **Timeline** — визуализация, какие тесты выполнялись параллельно и сколько заняли (полезно для оптимизации CI)
- **History/Trend** — динамика прохождения тестов между запусками (нужно хранить историю между CI-запусками, обычно копированием `history/` папки из предыдущего отчёта)

## Broken vs Failed — важное отличие (частый вопрос)

- **Failed** — тест упал именно на **assertion** (ожидаемое не совпало с фактическим) — это баг в приложении
- **Broken** — тест упал из-за **непредвиденного исключения** (NullPointerException, ElementNotFound, таймаут) — это может быть проблема самого теста/окружения, а не баг продукта

Allure автоматически разделяет эти статусы, что помогает быстро понять на дашборде: "это реальные баги" vs "надо чинить тесты".

## Частые вопросы на собеседовании

- _"Зачем нужен Allure, если есть стандартный Surefire report?"_ → наглядность, история трендов, шаги теста, вложения (скриншоты), группировка по бизнес-фичам, а не только по Java-классам — понятно не только разработчикам, но и менеджерам/QA lead
- _"Как ты будешь дебажить упавший тест в CI без доступа к машине, где он запускался?"_ → смотреть Allure-отчёт: шаги (`@Step`), приложенный скриншот, стектрейс, категорию (Failed/Broken)
- _"Чем `@Severity` полезен на практике?"_ → при массовом падении тестов после релиза сначала чинятся BLOCKER/CRITICAL тесты, а не в порядке алфавита

Хочешь разобрать ещё **интеграцию Allure с TestNG** (там чуть другой набор аннотаций, `@Step` vs listener-based подход) или пойдём дальше по теме — например, **Page Object Model** или **REST Assured** для API-тестирования на Java?