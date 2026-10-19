# CI/CD для Backend QA

## Оглавление
- [1. Термины CI и CD](#1-термины-ci-и-cd)
- [2. Pipeline, jobs, stages и runners](#2-pipeline-jobs-stages-и-runners)
- [3. Пример GitLab CI](#3-пример-gitlab-ci)
- [4. Артефакты, кэш, секреты и окружения](#4-артефакты-кэш-секреты-и-окружения)
- [5. Ошибки CI и работа QA](#5-ошибки-ci-и-работа-qa)

---

## 1. Термины CI и CD

**Continuous Integration** — частое объединение изменений с автоматической сборкой и проверками.

**Continuous Delivery** — выпуск в production технически подготовлен, но может требовать ручного одобрения.

**Continuous Deployment** — каждое изменение, прошедшее установленные автоматические проверки, выпускается без отдельного ручного решения.

Сокращение CD неоднозначно — уточняй, что имеет в виду команда.

~~~text
Push → Lint / Unit → Integration → Image → Deploy test → API smoke → Release
~~~

Этапы могут запускаться параллельно и в разных workflow.

## 2. Pipeline, jobs, stages и runners

- **Pipeline** — последовательность/граф заданий CI по событию, например push или merge request.
- **Job** — отдельная задача с командами.
- **Stage** — логическая группа задач.
- **Runner/Agent** — исполнитель job.
- **Artifact** — результат job для просмотра/передачи (Allure, test-results, coverage).
- **Cache** — повторно используемые файлы, ускоряющие сборку, но не замена artifact.

## 3. Пример GitLab CI

~~~yaml
stages:
  - test
  - build

unit-tests:
  stage: test
  image: node:22
  script:
    - npm ci
    - npm run test:unit

api-tests:
  stage: test
  image: node:22
  script:
    - npm ci
    - npm run test:api
  artifacts:
    when: always
    paths:
      - test-results/

build:
  stage: build
  script:
    - docker build -t bonus-api:$CI_COMMIT_SHA .
~~~

**Учебный пример:** команды test:unit/test:api должны существовать в package.json, а runner должен быть настроен для Docker build. Наличие docker CLI само по себе не гарантирует доступ к daemon.

## 4. Артефакты, кэш, секреты и окружения

**Environment variables** передают параметры и секреты в job. Секреты хранят в защищённых переменных/хранилищах, не в открытом YAML и не в логах.

**Quality Gate** — согласованные требования перед merge/release: успешные проверки, review, политики безопасности. Зелёный pipeline показывает только, что конкретные проверки прошли при заданных условиях.

**Deploy strategies:**
- **Rolling:** постепенно заменяем экземпляры сервиса.
- **Canary:** направляем часть трафика новой версии и наблюдаем метрики.
- **Blue/Green:** переключаем трафик между двумя окружениями.

**Rollback** приложения не всегда откатывает миграции БД или опубликованные события. Нужна совместимость схем и данных.

## 5. Ошибки CI и работа QA

| Симптом | Гипотезы | Что проверю |
|---|---|---|
| Build failed | Ошибка зависимостей, сборки, окружения | Логи job, commit, образ |
| API-тест вернул 503 | Сервис не поднялся, зависимость недоступна | Health, container logs |
| Test timeout | Race, сеть, медленный стенд | Trace, лимиты, загруженность |
| Flaky-test | Shared state, ожидания, конкуренция | Изоляция, повторы с диагностикой |
| Нет отчёта | Неверный путь artifacts | Настройки и директорию результатов |

**Повторные запуски не исправляют flaky-тест.** Нужно установить причину и только потом менять retries/timeout.

**Пример:** после merge успешно собрали Docker image, но API smoke падает. Проверяю digest/версию развернутого образа, конфигурацию, доступность PostgreSQL, логи сервиса и корректность тестовых данных.

**Интервью:** чем CI отличается от CD? Что такое job, stage, artifact? Почему зелёный pipeline ещё не гарантирует отсутствие багов?

[Далее — Docker](03-docker.md).