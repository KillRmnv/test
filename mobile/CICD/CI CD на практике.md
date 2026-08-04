# CI/CD на практике — Stages, Jobs, Runners, выбор ОС

Разберём не просто теорию "что такое CI/CD", а как это реально устроено внутри — раз ты уже понимаешь смысл, пойдём в механику.

## Базовая иерархия: Pipeline → Stage → Job → Step

```
Pipeline (весь процесс)
 └── Stage 1: build
      └── Job: build-app
 └── Stage 2: test
      └── Job: unit-tests
      └── Job: integration-tests   ← job'ы внутри одного stage выполняются ПАРАЛЛЕЛЬНО
 └── Stage 3: deploy
      └── Job: deploy-staging
```

### Pipeline

Весь процесс от коммита до деплоя — набор stages, которые выполняются последовательно (stage 2 не начнётся, пока не завершится stage 1, если явно не настроено иначе).

### Stage (этап)

Логическая группа jobs. Пример типичных stages:

- `build` — компиляция/сборка приложения
- `test` — запуск тестов (unit, integration, e2e)
- `deploy` — выкладка на окружение

**Ключевое правило**: stages выполняются **последовательно**, jobs внутри одного stage — **параллельно** (если ресурсы позволяют и нет явных зависимостей).

### Job (задача)

Конкретная единица работы внутри stage — например, "запустить unit-тесты" или "прогнать regression suite". У каждого job:

- свой набор команд (`script`)
- своё окружение (может быть свой Docker-образ)
- свои зависимости (`needs`/`dependencies`)
- может быть настроен на конкретный runner

### Step (шаг)

Отдельная команда внутри job (например, `npm install`, `npm test`, `npm run build`) — самая мелкая единица.

## Пример на GitHub Actions (частый стек)

```yaml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest        # ← выбор ОС здесь
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Build with Maven
        run: mvn clean compile

  unit-tests:
    needs: build                  # ← зависимость: сначала build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run unit tests
        run: mvn test

  e2e-tests:
    needs: build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chrome, firefox]   # ← matrix strategy — запуск в нескольких конфигурациях
    steps:
      - name: Run Selenium tests on ${{ matrix.browser }}
        run: mvn test -Dbrowser=${{ matrix.browser }}

  deploy:
    needs: [unit-tests, e2e-tests]  # ← deploy только если ОБА job'а прошли успешно
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # ← деплой только из main
    steps:
      - name: Deploy to staging
        run: echo "Deploying..."
```

## Пример на GitLab CI (тоже частый на проектах EPAM)

```yaml
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  image: maven:3.9-eclipse-temurin-17   # ← выбор окружения через Docker-образ
  script:
    - mvn clean compile

unit-tests:
  stage: test
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn test
  artifacts:
    reports:
      junit: target/surefire-reports/*.xml   # ← отчёты для GitLab UI

regression-tests:
  stage: test
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn test -Dsuite=regression
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'   # ← запускать только на main

deploy-staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
  environment:
    name: staging
  when: manual   # ← ручной запуск деплоя (кнопка в UI)
```

## Как выбирается ОС / окружение для пайплайна

Это ключевой вопрос, разберём подробно.

### Вариант 1: `runs-on` / `tags` — указание типа runner'а

**GitHub Actions:**

```yaml
runs-on: ubuntu-latest    # Linux
runs-on: windows-latest   # Windows
runs-on: macos-latest     # macOS
```

GitHub предоставляет **hosted runners** — виртуальные машины, которые поднимаются под каждый job и уничтожаются после его завершения (чистое окружение каждый раз).

**GitLab CI:**

```yaml
tags:
  - linux
  - docker
```

GitLab использует **tags** для маршрутизации job на конкретный runner (например, `windows-runner`, `macos-runner`), которые заранее зарегистрированы администратором.

### Вариант 2: Docker-образ — если runner сам по себе универсальный (Linux), а нужное окружение задаётся образом

```yaml
image: maven:3.9-eclipse-temurin-17   # конкретная версия Java внутри контейнера
```

Это самый гибкий способ — runner (машина) может быть любой Linux-машиной, а нужная версия языка/зависимостей приходит из Docker-образа. Это упрощает **воспроизводимость**: у всех разработчиков и в CI используется одинаковое окружение.

### Self-hosted vs Cloud (hosted) runners

||Hosted (облачные)|Self-hosted|
|---|---|---|
|Кто поддерживает|GitHub/GitLab|Сама компания|
|Гибкость окружения|Ограничена|Полный контроль (можно поставить специфичный софт, лицензии, доступ к внутренней сети)|
|Стоимость|Оплата по времени использования|Оплата инфраструктуры|
|Частый кейс в enterprise (как EPAM)|Для open-source/публичных репо|Часто self-hosted runners внутри корпоративной сети, доступ к внутренним сервисам/БД|

**Частый вопрос на собеседовании**: _"Почему компании используют self-hosted runners?"_ → доступ к закрытым внутренним ресурсам (БД, внутренние API), контроль над версиями софта, экономия при больших объёмах сборок, соответствие security-политикам компании.

### Matrix strategy — запуск на нескольких ОС/версиях одновременно

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    java: [11, 17, 21]
```

Это создаёт **декартово произведение** — 3 ОС × 3 версии Java = 9 параллельных job'ов. Частый кейс для проверки кросс-платформенной совместимости (актуально, если вспомнить твой опыт с Proton/Wine — похожая идея тестирования на разных средах, только в контексте CI).

## Дополнительные механики, которые могут спросить

### Artifacts

Файлы, которые job сохраняет и передаёт дальше по пайплайну (например, скомпилированный `.jar`, отчёт тестов, скриншоты упавших тестов) — следующий stage может их скачать.

### Cache

В отличие от artifacts, cache используется для ускорения повторных запусков (например, кэш зависимостей Maven `~/.m2`, чтобы не скачивать их заново каждый раз).

### Triggers / Rules / Conditions

Когда job вообще должен запускаться:

- по push в конкретную ветку
- по pull/merge request
- по расписанию (`schedule`, cron)
- вручную (`when: manual`)
- при тэге релиза

### Parallel vs Sequential jobs, `needs`/`dependencies`

По умолчанию job'ы одного stage не знают друг о друге и идут параллельно. Если нужна зависимость внутри stage (например, e2e-тесты должны ждать, пока прогонятся unit-тесты, хотя формально они в одном stage) — используется `needs` (GitHub Actions) или `needs`/`dependencies` (GitLab).

## Частые практические вопросы на собеседовании

- _"Что произойдёт, если один job в stage упадёт?"_ → зависит от настройки: по умолчанию весь pipeline помечается failed, но можно настроить `allow_failure: true` (GitLab) или `continue-on-error: true` (GitHub), чтобы не блокировать остальные jobs
- _"Как ты бы настроил, чтобы regression-тесты гонялись только по ночам, а smoke — на каждый коммит?"_ → `schedule`/cron trigger для regression в отдельном job, а smoke-тесты — в основном pipeline на каждый push
- _"Зачем нужен отдельный stage для build, если можно сразу тестировать?"_ → чтобы не тратить время на прогон тестов, если приложение даже не собралось — fail fast принцип

Хочешь, разберём отдельно **Jenkins** (Jenkinsfile, declarative vs scripted pipeline) — он тоже часто встречается на enterprise-проектах и может отдельно спрашиваться на собеседовании?