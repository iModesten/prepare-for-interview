# TypeScript для автоматизации QA

## Оглавление
- [1. Типы и интерфейсы](#1-типы-и-интерфейсы)
- [2. Promise и async/await](#2-promise-и-asyncawait)
- [3. Ошибки и контракт API](#3-ошибки-и-контракт-api)
- [4. Тестовые данные](#4-тестовые-данные)
- [5. Вопросы](#5-вопросы)

---

## 1. Типы и интерфейсы

TypeScript добавляет статическую проверку типов. Во время выполнения JavaScript типы TypeScript не защищают входящие данные от ошибок.

~~~ts
type Status = 'available' | 'issued' | 'expired';
interface Bonus {
  id: string;
  status: Status;
  amount: number;
}
function isAvailable(b: Bonus) { return b.status === 'available'; }
~~~

**type vs interface:** interface удобно для формы объектов, type — для unions и композиции; есть пересечения в возможностях.

**unknown vs any:** unknown требует проверить тип до использования, any снимает проверку. Приведение типа через as не проверяет JSON в runtime. Для контракта API полезна схема или явная проверка структуры.

## 2. Promise и async/await

~~~ts
async function loadBonus(id: string): Promise<Bonus> {
  const response = await fetch('/api/bonuses/' + id);
  if (!response.ok) throw new Error('HTTP ' + response.status);
  return await response.json() as Bonus;
}
~~~

**Promise** — представление будущего результата. **async**-функция возвращает Promise. **await** приостанавливает её выполнение до результата или ошибки. **Promise.all** полезен для независимых параллельных операций.

Важная ошибка Playwright:

~~~ts
// Плохо: асинхронный assertion не дождались
expect(page.getByText('Выдан')).toBeVisible();

// Правильно
await expect(page.getByText('Выдан')).toBeVisible();
~~~

## 3. Ошибки и контракт API

HTTP 500 — полученный HTTP-ответ с ошибкой сервера; ECONNREFUSED — сетевой отказ до HTTP-ответа.

~~~ts
const result = await request.get('/api/bonuses');
expect(result.ok()).toBeTruthy();
const body: unknown = await result.json();
expect(Array.isArray(body)).toBeTruthy();
~~~

Проверка Array.isArray подтверждает только массив, а не типы полей каждого элемента.

Используй try/finally для очистки ресурсов и тестовых данных после упавшей проверки. Не скрывай ошибки catch-блоком без отчёта о проблеме.

## 4. Тестовые данные

Генерируй уникальные userId/bonusId для параллельных тестов. Учитывай: данные должны быть воспроизводимыми и очищаемыми, уникальный UUID не заменяет правильную бизнес-подготовку.

~~~ts
import { randomUUID } from 'node:crypto';
const testUserId = randomUUID();

const apiUrl = process.env.API_URL;
if (!apiUrl) throw new Error('API_URL not set');
~~~

**null vs undefined:** null — явное отсутствие значения, undefined — не определено. Оператор ?? подставляет значение только при null/undefined, а || — при любом falsy значении, включая 0.

## 5. Вопросы

Чем unknown лучше any? Почему опасно приведение типа ответа API? Что будет, если забыть await? Когда Promise.all нежелателен? Как гарантировать очистку ресурсов?

[Дальше — Playwright](02-playwright-fundamentals.md).