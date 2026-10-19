# Kubernetes — основы для Backend QA

## Оглавление
- [1. Зачем нужен Kubernetes](#1-зачем-нужен-kubernetes)
- [2. Pod, Deployment и Service](#2-pod-deployment-и-service)
- [3. ConfigMap, Secret, Namespace](#3-configmap-secret-namespace)
- [4. Probes, ресурсы и обновления](#4-probes-ресурсы-и-обновления)
- [5. kubectl для QA](#5-kubectl-для-qa)
- [6. Практический кейс](#6-практический-кейс)

---

## 1. Зачем нужен Kubernetes

**Kubernetes** — платформа оркестрации контейнеризированных приложений: размещает экземпляры, следит за желаемым количеством реплик, обеспечивает сетевой доступ и обновления.

Kubernetes не устраняет автоматически ошибки приложения, несогласованные версии API или проблемы БД.

~~~text
Developer → Docker image → Registry
                              ↓
                        Kubernetes Cluster
                           Deployment
                         /      |      \
                       Pod A  Pod B   Pod C
                         \      |      /
                           Service
                              ↓
                           Client
~~~

## 2. Pod, Deployment и Service

**Pod** — минимальная единица размещения в Kubernetes. Содержит один или несколько контейнеров, разделяющих сеть и некоторые ресурсы. IP Pod может меняться при пересоздании.

**Deployment** описывает желаемое состояние набора Pod: число реплик, шаблон контейнера, обновление. Обычно управляет ReplicaSet.

**Service** — стабильная точка доступа к набору Pod, найденных по labels/selectors. Service не гарантирует исправность бизнес-логики.

**Ingress** — набор правил для входящего HTTP(S)-трафика, реализуемый Ingress Controller. Возможна альтернатива через Gateway API или внешние механизмы.

**Пример:** если один Pod бонусного сервиса завершился, Deployment стремится восстановить число реплик. Но если новая версия падает в каждом Pod, одни только рестарты не помогут.

## 3. ConfigMap, Secret, Namespace

- **Namespace** — логическая область имён ресурсов внутри кластера.
- **ConfigMap** — обычная конфигурация (имена зависимостей, параметры поведения).
- **Secret** — ресурс для чувствительных данных. **Не означает, что секрет автоматически надёжно зашифрован на всех этапах**: важны настройки хранения, RBAC и доступа.
- **Labels / selectors** — метки и правила отбора, связывающие Service с Pod.

**Практика:** тестовый API стучится не в ту PostgreSQL из-за неверного ConfigMap. QA проверяет адреса, Namespace, версию приложения, но не копирует значения секретов в баг-репорт.

## 4. Probes, ресурсы и обновления

| Probe | Что означает |
|---|---|
| **Liveness** | Нужно ли перезапустить контейнер при застревании? |
| **Readiness** | Готов ли Pod принимать обычный трафик? |
| **Startup** | Дать медленно стартующему приложению время до применения других probes |

Если readiness не проходит, Pod обычно убирается из endpoint-списка соответствующего Service. Это отличается от рестарта по liveness.

**Requests** — ресурсы, учитываемые при планировании. **Limits** — ограничения; превышение лимита памяти может привести к OOMKilled. CPU throttling возможен при CPU limit.

**Rolling Update** постепенно заменяет Pod. При ошибке может понадобиться rollback Deployment, но откат Pod **не откатывает автоматически миграцию БД**.

## 5. kubectl для QA

| Цель | Команда |
|---|---|
| Список Pod | `kubectl get pods -n test` |
| Deployment и состояние | `kubectl get deployments -n test` |
| Service | `kubectl get svc -n test` |
| Посмотреть Pod | `kubectl describe pod <pod> -n test` |
| Последние логи | `kubectl logs <pod> -n test --tail=100` |
| Логи предыдущего рестарта | `kubectl logs <pod> -n test --previous` |
| События Namespace | `kubectl get events -n test --sort-by=.lastTimestamp` |
| Состояние rollout | `kubectl rollout status deployment/bonus-api -n test` |
| Временно пробросить порт | `kubectl port-forward service/bonus-api 8080:80 -n test` |

Команды требуют настроенного kubectl context и прав. **Перед любым изменяющим действием проверь текущий context и namespace**, чтобы не менять production случайно.

## 6. Практический кейс

**После деплоя API возвращает 503.**

1. `kubectl get pods -n test` — есть ли Running/Ready Pod?
2. `kubectl describe pod ...` — ошибки запуска, probes, ImagePullBackOff?
3. `kubectl logs ...` и при перезапусках `--previous` — ошибка конфигурации или приложения?
4. `kubectl get svc -n test` — корректен ли Service?
5. Проверить, направляет ли Service запросы на готовые Pod, не сломан ли маршрут на Ingress/Gateway.
6. После исправления провести smoke API и связанных интеграций.

**CrashLoopBackOff** — повторяющиеся падения контейнера с задержкой между стартами. **ImagePullBackOff** — не удаётся получить образ. **Pending** — Pod пока не размещён/не готов к запуску: причины бывают разные, смотри events.

**Интервью:** Pod vs Container? Для чего Deployment? Почему Pod Running, но не Ready? Что происходит при liveness failure? Почему pod IP не стоит зашивать в конфигурацию?

[Далее — вопросы для собеседования](05-devops-interview-questions.md).