# API Automation, Test Data и архитектура Playwright

## Оглавление
- [1. APIRequestContext](#1-apirequestcontext)
- [2. API-сценарии](#2-api-сценарии)
- [3. Mock, Stub и Spy](#3-mock-stub-и-spy)
- [4. Page Object, Component Object](#4-page-object-component-object)
- [5. Flaky и дизайн тестового проекта](#5-flaky-и-дизайн-тестового-проекта)

---

## 1. APIRequestContext

Playwright поддерживает HTTP-запросы без браузерного UI через встроенную test fixture request.

~~~ts
import { test, expect } from '@playwright/test';

test('возвращает список бонусов', async ({ request }) => {
  const response = await request.get('/api/bonuses');
  expect(response.status()).toBe(200);
  const body = await response.json();
  expect(Array.isArray(body)).toBeTruthy();
});
~~~

Для относительных URL настрой baseURL. Не путай request fixture с запросами страницы: существуют разные APIRequestContext и разные правила cookies.

## 2. API-сценарии

Для выдачи бонуса проверяем контракт успеха/ошибок, права, идемпотентность, параллелизм, запись в БД, Kafka. Не обязательно повторять все unit-тесты разработчика.

~~~ts
test('повтор выдачи не нарушает лимит', async ({ request }) => {
  const id = 'unique-test-id'; // Сгенерировать уникальные данные
  const headers = { 'Idempotency-Key': id };
  const data = { userId: 'user-test', bonusId: 'bonus-test' };

  const first = await request.post('/api/bonuses/issue', { data, headers });
  const second = await request.post('/api/bonuses/issue', { data, headers });

  expect(first.ok()).toBeTruthy();
  // Проверка статуса второго ответа зависит от контракта.
  // ОБЯЗАТЕЛЬНО отдельно проверить, что в системе ровно одна выдача.
  expect(second.status()).toBeLessThan(500);
});
~~~

Это **шаблон**, не полноценный тест бизнес-инварианта без дополнительной проверки БД/истории. Случайный ключ должен быть отдельным на тест и повторно использоваться в рамках одного сценария.

## 3. Mock, Stub и Spy

**Stub** отдаёт заранее заданный результат; **Mock** подменяет зависимость, иногда проверяет взаимодействия; **Spy** фиксирует обращения. Термины различаются по библиотекам.

~~~ts
await page.route('**/api/bonuses', route =>
  route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([{ id: 'bonus-test', status: 'available' }]),
  })
);
~~~

Это UI-проверка со стабом API, **не** проверка настоящей интеграции UI и Backend. Для контракта нужны независимые API/integration проверки.

## 4. Page Object, Component Object

Page Object скрывает реализацию UI-действий и локаторы. Component Object описывает повторно используемые виджеты.

~~~ts
import type { Page } from '@playwright/test';
export class BonusesPage {
  constructor(private readonly page: Page) {}
  async open() { await this.page.goto('/bonuses'); }
  get issueButton() {
    return this.page.getByRole('button', { name: 'Выдать' });
  }
}
~~~

Избегай огромных Page Object, знающих одновременно UI, Kafka, БД и все бизнес-правила. Assertions должны оставаться понятными из сценария теста.

## 5. Flaky и дизайн тестового проекта

~~~text
tests/api/bonus-issue.spec.ts
tests/ui/bonuses.spec.ts
clients/bonus-api.ts
fixtures/users.ts
factories/bonus.ts
pages/bonuses.page.ts
~~~

**Частые причины flaky:** shared mutable state, фиксированный sleep, нестабильные локаторы, ошибки синхронизации, внешние зависимости.

Рекомендации:
- независимые пользователи и данные на тест/worker;
- ожидания условий вместо ожиданий миллисекунд;
- очистка данных или изолированные стенды;
- trace и логи на падениях;
- покрытие по рискам, не по количеству тестов.

**Code review:** тест должен проверять нужный контракт, не скрывать ошибки, читаться, корректно ждать и завершать операции, не раскрывать секреты.

[Дальше — вопросы](04-automation-interview-questions.md).