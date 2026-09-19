# API — черновик

Это не финальная спецификация. Не контракт.
Просто набросок: как **могли бы** выглядеть endpoints,
если кто-то решит реализовать идею.

Всё мягко. Всё можно менять.

---

## Общие принципы

- **REST** — просто и понятно.
- **JSON** — для запросов и ответов.
- **JWT** — для аутентификации.
- **HTTPS** — обязательно.
- **Версионирование:** `/api/v1/...`

---

## Аутентификация

| Метод | Endpoint | Описание |
|---|---|---|
| POST | `/api/v1/auth/register` | Регистрация |
| POST | `/api/v1/auth/login` | Вход |
| POST | `/api/v1/auth/logout` | Выход |
| POST | `/api/v1/auth/refresh` | Обновление токена |
| GET | `/api/v1/auth/me` | Текущий пользователь |
| POST | `/api/v1/auth/gosuslugi` | Вход через Госуслуги |

---

## Профили

| Метод | Endpoint | Описание |
|---|---|---|
| GET | `/api/v1/profile` | Свой профиль |
| PATCH | `/api/v1/profile` | Обновить профиль |
| POST | `/api/v1/profile/diploma` | Загрузить диплом |
| POST | `/api/v1/profile/self-employed` | Указать НПД |
| GET | `/api/v1/profile/payouts` | История выплат |

---

## Статьи

| Метод | Endpoint | Описание |
|---|---|---|
| GET | `/api/v1/articles` | Список статей |
| GET | `/api/v1/articles/:id` | Одна статья |
| POST | `/api/v1/articles` | Создать статью |
| PATCH | `/api/v1/articles/:id` | Редактировать |
| DELETE | `/api/v1/articles/:id` | Удалить (свою) |
| POST | `/api/v1/articles/:id/publish` | Отправить на модерацию |
| GET | `/api/v1/articles/:id/likes` | Лайки статьи |

---

## Лайки

| Метод | Endpoint | Описание |
|---|---|---|
| POST | `/api/v1/articles/:id/like` | Обычный лайк |
| POST | `/api/v1/articles/:id/expert-like` | Экспертный лайк (100 ₽) |
| DELETE | `/api/v1/articles/:id/like` | Убрать лайк |
| GET | `/api/v1/likes/my` | Мои лайки (эксперт) |

**Особенности:**
- Экспертный лайк проверяет геофильтр и аффилиацию.
- Если проверка не прошла — ошибка 403.
- Если баланс карты < 100 — ошибка 402.

---

## Экспертная карта

| Метод | Endpoint | Описание |
|---|---|---|
| GET | `/api/v1/expert/card` | Баланс и история |
| GET | `/api/v1/expert/card/transactions` | Транзакции |
| POST | `/api/v1/expert/card/issue` | Выпуск (если нужно) |

---

## Выплаты

| Метод | Endpoint | Описание |
|---|---|---|
| GET | `/api/v1/payouts` | История выплат |
| POST | `/api/v1/payouts/request` | Запрос на вывод |
| GET | `/api/v1/payouts/:id` | Статус выплаты |
| GET | `/api/v1/payouts/balance` | Текущий баланс |

**Особенности:**
- Порог вывода: например, от 500 ₽.
- Налог 6% удерживается автоматически.
- Чек формируется через «Мой налог».

---

## Модерация

| Метод | Endpoint | Описание |
|---|---|---|
| GET | `/api/v1/moderation/articles` | Очередь статей |
| POST | `/api/v1/moderation/articles/:id/approve` | Одобрить |
| POST | `/api/v1/moderation/articles/:id/reject` | Отклонить |
| GET | `/api/v1/moderation/users` | Очередь пользователей |
| POST | `/api/v1/moderation/users/:id/verify` | Верифицировать |
| POST | `/api/v1/moderation/reports` | Жалоба |

---

## Уведомления

| Метод | Endpoint | Описание |
|---|---|---|
| GET | `/api/v1/notifications` | Список уведомлений |
| POST | `/api/v1/notifications/read` | Отметить прочитанным |
| PATCH | `/api/v1/notifications/settings` | Настройки |

---

## Внешние интеграции (внутренние endpoints)

| Метод | Endpoint | Описание |
|---|---|---|
| POST | `/api/v1/integrations/gosuslugi/callback` | Callback от Госуслуг |
| POST | `/api/v1/integrations/frdo/verify` | Проверка диплома через ФРДО |
| POST | `/api/v1/integrations/moy-nalog/receipt` | Формирование чека |
| POST | `/api/v1/integrations/sbp/payout` | Выплата через СБП |

---

## Примеры запросов и ответов

### POST `/api/v1/auth/login`

**Запрос:**
```json

{
  "email": "user@example.com",
  "password": "••••••••"
}

**Ответ:**
```json

{
  "access_token": "eyJhbGciOi...",
  "refresh_token": "eyJhbGciOi...",
  "user": {
    "id": 1,
    "email": "user@example.com",
    "role": "author"
  }
}

POST /api/v1/articles/:id/expert-like
**Ответ (успех):**
```json

{
  "status": "ok",
  "amount": 100,
  "author_id": 42,
  "card_balance": 4900,
  "message": "Лайк учтён. Автор получит 100 ₽."
}

**Ответ (ошибка — геофильтр):**

{
  "status": "error",
  "code": 403,
  "message": "Нельзя лайкать авторов из своего региона."
}

**Ответ (ошибка — баланс карты):**

{
  "status": "error",
  "code": 402,
  "message": "Недостаточно средств на экспертной карте."
}

GET /api/v1/payouts/balance
**Ответ:**

{
  "gross": 2700,
  "tax": 162,
  "net": 2538,
  "status": "available",
  "min_payout": 500
}

**Коды ошибок**
Код	Значение
200	OK
201	Создано
400	Неверный запрос
401	Не авторизован
403	Запрещено (геофильтр, аффилиация)
402	Недостаточно средств
404	Не найдено
409	Конфликт (двойной лайк)
429	Слишком много запросов
500	Ошибка сервера
Что важно помнить
1 эксперт = 1 лайк на статью.

Геофильтр и аффилиация — обязательны.

Налог 6% — автоматически.

Порог вывода — обсуждается.

Безопасность — прежде всего.

Чего здесь нет
❌ Финальной спецификации.

❌ OpenAPI / Swagger (пока).

❌ Гарантий, что endpoints не изменятся.

❌ Обязательств реализовать именно так.

Только черновик. Можно делать иначе. 🙂

---
