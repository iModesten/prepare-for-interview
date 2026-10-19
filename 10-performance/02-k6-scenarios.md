# k6: сценарии, executors, thresholds и checks

## Оглавление
- [1. Базовый скрипт k6](#1-базовый-скрипт-k6)
- [2. Closed vs Open Workload](#2-closed-vs-open-workload)
- [3. Executors и stages](#3-executors-и-stages)
- [4. Checks, thresholds и tags](#4-checks-thresholds-и-tags)
- [5. Командная строка](#5-командная-строка)

---

## 1. Базовый скрипт k6

k6 исполняет JavaScript-тестовые сценарии. **Классический runtime k6 не запускает произвольный Node.js-код**, поэтому не стоит переносить Node-зависимости без проверки совместимости. Существуют поддерживаемые способы использования TypeScript посредством сборки; версия и toolchain важны.

~~~js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 10,
  duration: '30s',
  thresholds: {
    http_req_failed: ['rate<0.01'],
    http_req_duration: ['p(95)<500'],
  },
};

export default function () {
  const res = http.get(__ENV.BASE_URL + '/api/bonuses');
  check(res, { 'status is 200': r => r.status === 200 });
  sleep(1);
}
~~~

**Важно:** BASE_URL передаётся через окружение, например BASE_URL=https://test.example.com. Запускай только на разрешённом стенде.

## 2. Closed vs Open Workload

**Closed model:** число активных VU задано; следующий запрос пользователя зависит от завершения предыдущего. При замедлении системы итоговый RPS часто падает.

**Open model:** задаётся интенсивность поступления итераций независимо от времени ответа. k6 создаёт достаточно VU в пределах настроенных ограничений, иначе возникают dropped_iterations.

**Пример:** если тест должен воспроизводить 100 новых операций в секунду, чаще подходит arrival-rate executor, а не попытка угадать точное число VU.

**Важное различие:** k6 считает **iterations**, а не обязательно HTTP-запросы. Если в одной итерации 3 HTTP-вызова, 10 iterations/s способны породить около 30 запросов/s.

## 3. Executors и stages

**constant-vus** — фиксированное число VU.  
**ramping-vus** — VU меняются по этапам.  
**constant-arrival-rate** — фиксированная скорость старта итераций.  
**ramping-arrival-rate** — скорость старта итераций меняется по этапам.

~~~js
export const options = {
  scenarios: {
    bonus_list: {
      executor: 'constant-arrival-rate',
      rate: 20,              // 20 итераций / timeUnit
      timeUnit: '1s',
      duration: '5m',
      preAllocatedVUs: 30,
      maxVUs: 100,
    },
  },
};
~~~

**preAllocatedVUs** — заранее выделенные исполнители; **maxVUs** — верхний предел для расширения. Они не задают RPS напрямую. Если VU не хватает, k6 может не запустить часть итераций и учесть их в dropped_iterations.

**Ramping пример:**

~~~js
export const options = {
  scenarios: {
    load: {
      executor: 'ramping-arrival-rate',
      startRate: 5,
      timeUnit: '1s',
      preAllocatedVUs: 30,
      maxVUs: 150,
      stages: [
        { target: 20, duration: '2m' },
        { target: 50, duration: '3m' },
        { target: 5, duration: '1m' },
      ],
    },
  },
};
~~~

## 4. Checks, thresholds и tags

**Check** — записывает успешность проверки, но сам по себе **не обязан провалить весь прогон**.

**Threshold** — проверяемый критерий метрики; его нарушение влияет на итоговый статус теста.

~~~js
import { check } from 'k6';
check(res, { 'valid status': r => r.status === 200 });
~~~

~~~js
export const options = {
  thresholds: {
    'http_req_duration{endpoint:list}': ['p(95)<500'],
    'http_req_failed': ['rate<0.01'],
    'checks': ['rate>0.99'],
  },
};
~~~

Для группировки метрик передавай tags:

~~~js
http.get(__ENV.BASE_URL + '/api/bonuses', {
  tags: { endpoint: 'list' },
});
~~~

Если отдельные методы имеют разные нагрузки, нужна отдельная проверка каждого endpoint.

## 5. Командная строка

~~~bash
BASE_URL=https://test.example.com k6 run script.js
k6 run --summary-export=summary.json script.js
~~~

**Безопасность:** нагрузка способна изменять реальные данные и мешать пользователям. На production только по заранее согласованному плану с ограничением нагрузки и monitoring.

**Интервью:** почему «50 VU» не означает «50 RPS»? Чему равен rate — HTTP-запросам или итерациям? Что сигнализирует dropped_iterations?

[Далее — проектирование нагрузки](03-load-profiles-and-analysis.md).