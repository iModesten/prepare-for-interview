# Playwright Test для Backend и WebView QA

## Оглавление
- [1. Browser, Context, Page](#1-browser-context-page)
- [2. Locators и ожидания](#2-locators-и-ожидания)
- [3. Fixtures и Hooks](#3-fixtures-и-hooks)
- [4. Запуск, retries, parallel](#4-запуск-retries-parallel)
- [5. Debugging и trace](#5-debugging-и-trace)

---

## 1. Browser, Context, Page

**Playwright** управляет браузерами; **Playwright Test** добавляет runner, fixtures, assertions и отчёты.

- **Browser:** процесс браузера.
- **BrowserContext:** изолированная сессия с собственными cookies/storage.
- **Page:** вкладка внутри контекста.
- **Fixture:** ресурс с управляемым жизненным циклом, например page, request или собственный API client.

Обычно test-scoped page создаётся с новым изолированным context для каждого теста. Общие cookies или глобальный тестовый пользователь могут сделать тесты зависимыми от порядка.

## 2. Locators и ожидания

~~~ts
import { test, expect } from '@playwright/test';

test('показывает доступный бонус', async ({ page }) => {
  await page.goto('/bonuses');
  await page.getByRole('button', { name: 'Мои бонусы' }).click();
  await expect(page.getByText('Доступен')).toBeVisible();
});
~~~

**Locator** повторно находит элемент во время действий/проверок. Playwright ждёт подходящие условия для выполнения действия (actionability).

Обычно предпочтительны getByRole, getByLabel, getByTestId, а не длинные CSS/XPath, завязанные на структуру DOM.

**Retrying assertion**: await expect(locator).toBeVisible() ждёт условие до таймаута. Синхронный expect(value).toEqual() просто сравнивает значение.

~~~ts
const resultPromise = page.waitForResponse(r =>
  r.url().includes('/api/bonuses') && r.status() === 200
);
await page.getByRole('button', { name: 'Обновить' }).click();
const response = await resultPromise;
~~~

Ожидание нужно установить до действия, иначе можно пропустить быстрый ответ. Не подменяй событие произвольным waitForTimeout(5000).

## 3. Fixtures и Hooks

~~~ts
import { test as base, expect } from '@playwright/test';

type Fixtures = { bonusId: string };
export const test = base.extend<Fixtures>({
  bonusId: async ({ request }, use) => {
    // Условный API подготовки, адаптировать под свой стенд
    const response = await request.post('/test/bonuses');
    const { id } = await response.json();
    try {
      await use(id);
    } finally {
      await request.delete('/test/bonuses/' + id);
    }
  },
});
export { expect };
~~~

Это **иллюстрация**, endpoints подготовки не стандартизованы. В реальном проекте обрабатывай ошибки setup/cleanup и защищай от удаления чужих данных.

**beforeEach** — запуск перед тестом; **beforeAll** — перед набором в worker. Общие ресурсы из beforeAll не стоит изменять параллельными тестами.

## 4. Запуск, retries, parallel

~~~ts
import { defineConfig } from '@playwright/test';
export default defineConfig({
  use: { baseURL: 'http://localhost:3000', trace: 'retain-on-failure' },
  retries: process.env.CI ? 1 : 0,
  workers: process.env.CI ? 2 : undefined,
});
~~~

~~~bash
npx playwright test
npx playwright test --headed
npx playwright test --debug
npx playwright show-report
~~~

**Retries не лечат flaky tests**. Параллельность важна для скорости, но требует изоляции данных.

## 5. Debugging и trace

При падении ищем конкретный failed assertion, trace, сетевые ответы, HTML/DOM, screenshot и версию браузера. Trace сохраняем в CI artifacts, учитывая, что в нём могут оказаться cookies, токены и личные данные.

**Вопрос:** почему тесты с фиксированными паузами flaky? Время ответа изменчиво, а сон не ждёт конкретное состояние.

[Дальше — API-тесты](03-api-and-test-architecture.md).