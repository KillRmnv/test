# Чем Jenkins уникален на фоне GitHub Actions / GitLab CI

## Главное отличие: архитектура Master-Agent (Controller-Agent)

Это ключевая особенность, которую стоит знать в первую очередь.

```
Jenkins Controller (Master)
    │
    ├── Agent 1 (Linux)
    ├── Agent 2 (Windows)
    └── Agent 3 (macOS)
```

- **Controller (Master)** — центральный сервер, который управляет пайплайнами, хранит конфигурацию, показывает UI, но **сам обычно не выполняет тяжёлую работу** (сборку/тесты)
- **Agent (Node/Worker)** — отдельная машина (физическая, виртуальная, Docker-контейнер), которая реально выполняет job'ы

В отличие от GitHub Actions/GitLab, где "runner" — просто исполнитель, в Jenkins это полноценная **распределённая система**, которую ты сам разворачиваешь и администрируешь (в этом и разница — Jenkins **self-hosted by default**, у него нет "облачного" аналога hosted runners от GitHub).

## Выбор ОС/окружения в Jenkins — через `agent` и `label`

```groovy
pipeline {
    agent none   // не задаём агента на уровне всего pipeline

    stages {
        stage('Build') {
            agent { label 'linux' }   // ← этот stage пойдёт на agent с меткой "linux"
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Test on Windows') {
            agent { label 'windows' }   // ← а этот — на agent с меткой "windows"
            steps {
                bat 'mvn test'   // bat вместо sh — Windows-команда
            }
        }
        stage('Test in Docker') {
            agent {
                docker { image 'maven:3.9-eclipse-temurin-17' }   // ← или через Docker-образ
            }
            steps {
                sh 'mvn test'
            }
        }
    }
}
```

**Механика**: администратор заранее регистрирует agent'ов в Jenkins (физические машины, VM, Docker-хосты) и присваивает им **labels** (`linux`, `windows`, `docker`, `gpu` и т.д.). Pipeline сам не знает, что за машина — он просто просит "дай мне agent с меткой X", и Jenkins сам находит свободный подходящий agent из пула. Это очень похоже на `tags` в GitLab, но more manual — ты сам управляешь пулом машин.

## Declarative vs Scripted Pipeline — уникальная особенность Jenkins

GitHub Actions и GitLab используют декларативный YAML без альтернатив. **Jenkins даёт выбор**:

### Declarative (современный, рекомендуемый стиль)

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
}
```

### Scripted (старый стиль, чистый Groovy)

```groovy
node {
    stage('Test') {
        sh 'mvn test'
    }
}
```

**Ключевое отличие**: Scripted Pipeline — это **полноценный Groovy-скрипт** с циклами, условиями, try/catch, произвольной логикой — гораздо более гибкий, но и сложнее поддерживать. Declarative — более строгая структура (легче читать, валидировать), но менее гибкая "из коробки" (хотя внутри `script {}` блока можно вставлять Groovy-код и в declarative).

Частый вопрос на собеседовании: _"Почему declarative считается лучшей практикой?"_ → более строгий синтаксис, встроенная валидация структуры, легче для новичков в команде, лучше интегрируется с Jenkins UI (визуализация стадий из коробки).

## Groovy как язык конфигурации — тоже уникально

GitHub Actions/GitLab CI — декларативный YAML. Jenkinsfile — это **Groovy DSL**, что даёт мощь полноценного языка программирования прямо в пайплайне:

```groovy
def browsers = ['chrome', 'firefox', 'edge']
for (browser in browsers) {
    stage("Test on ${browser}") {
        sh "mvn test -Dbrowser=${browser}"
    }
}
```

Такое сложнее сделать нативно в YAML-based системах (там для этого нужен `matrix` как отдельная фича, а в Jenkins это просто цикл на Groovy).

## Огромная экосистема плагинов

Это то, чем Jenkins исторически выделяется — **тысячи плагинов** для интеграции почти с чем угодно (Slack, Jira, Docker, Kubernetes, всевозможные тест-репортеры, security-сканеры). GitHub Actions/GitLab тоже имеют marketplace, но Jenkins на рынке дольше всех, и его плагинная экосистема — самая зрелая (и самая "болезненная" в поддержке — плагины могут ломаться друг о друга при обновлениях, это частая головная боль в реальных проектах).

## Blue Ocean — визуальный UI (не обязательно, но стоит знать)

Отдельный плагин/UI для Jenkins, дающий более современную визуализацию пайплайна (похожую на GitHub Actions), потому что классический Jenkins UI считается устаревшим и не очень user-friendly.

## Сравнительная таблица для собеседования

||Jenkins|GitHub Actions|GitLab CI|
|---|---|---|---|
|Хостинг|Self-hosted (ты сам разворачиваешь)|Cloud (hosted) + self-hosted runners|Cloud (hosted) + self-hosted runners|
|Конфигурация|Groovy (Jenkinsfile)|YAML|YAML|
|Гибкость логики|Высокая (полноценный язык)|Ограничена YAML + expressions|Ограничена YAML + expressions|
|Выбор окружения|`agent`/`label` (сам управляешь пулом)|`runs-on` (готовые VM от GitHub)|`tags`/`image`|
|Порог входа|Выше (нужно администрировать сервер)|Ниже (всё готово из коробки)|Ниже|
|Частый кейс в Enterprise|Legacy-проекты, сложные custom pipeline, полный контроль над инфраструктурой|Открытые/облачные проекты|Проекты на GitLab (self-hosted GitLab часто в enterprise)|

## Частый вопрос на собеседовании

_"Почему на legacy Java-проектах в энтерпрайзе (как EPAM) часто именно Jenkins, а не GitHub Actions?"_ → Jenkins существует с 2011 года (изначально Hudson), у многих компаний уже есть готовая инфраструктура, self-hosted даёт полный контроль над данными (важно для банков/страховых с compliance-требованиями), плюс исторически сложившаяся экспертиза внутри команд.

Хочешь, разберём ещё **Jenkinsfile для Java/Maven-проекта с интеграцией JUnit-отчётов и Selenium** — то, что реально может встретиться на практике при твоём стеке?