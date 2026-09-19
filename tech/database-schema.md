# Схема базы данных — возможный вариант

Это не финальная схема. Не обязательная.
Просто набросок: как **могли бы** выглядеть таблицы,
если кто-то решит реализовать идею.

Всё мягко. Всё можно менять.

---

## Основные сущности

1. **Пользователи** (users)
2. **Роли** (roles)
3. **Профили авторов** (author_profiles)
4. **Профили экспертов** (expert_profiles)
5. **Статьи** (articles)
6. **Лайки** (likes)
7. **Экспертные карты** (expert_cards)
8. **Транзакции** (transactions)
9. **Выплаты** (payouts)
10. **Модерация** (moderation_logs)

Плюс — вспомогательные таблицы.

---

## 1. users — Пользователи

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| email | varchar, unique | Email |
| password_hash | varchar | Хеш пароля |
| role | varchar | reader / author / expert / moderator / admin |
| created_at | timestamp | Дата регистрации |
| updated_at | timestamp | Дата обновления |
| is_active | boolean | Активен ли |
| is_verified | boolean | Верифицирован ли |

---

## 2. author_profiles — Профили авторов

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| user_id | bigint, FK | Ссылка на users |
| diploma_scan_url | varchar | Ссылка на скан диплома |
| diploma_number | varchar | Номер диплома |
| diploma_verified | boolean | Проверен ли диплом |
| self_employed | boolean | Есть ли статус НПД |
| self_employed_inn | varchar | ИНН самозанятого |
| orcid | varchar | ORCID (по желанию) |
| affiliation | varchar | Аффилиация |
| region | varchar | Регион (для геофильтра) |
| created_at | timestamp | Дата создания |

---

## 3. expert_profiles — Профили экспертов

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| user_id | bigint, FK | Ссылка на users |
| institution | varchar | Институт / вуз |
| position | varchar | Должность (профессор и т.д.) |
| region | varchar | Регион |
| verified | boolean | Верифицирован ли |
| card_id | bigint, FK | Ссылка на expert_cards |
| created_at | timestamp | Дата создания |

---

## 4. articles — Статьи

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| author_id | bigint, FK | Ссылка на author_profiles |
| title | varchar | Заголовок |
| abstract | text | Аннотация |
| keywords | varchar[] | Ключевые слова |
| file_url | varchar | Ссылка на PDF |
| status | varchar | draft / moderation / published / blocked |
| published_at | timestamp | Дата публикации |
| created_at | timestamp | Дата создания |
| updated_at | timestamp | Дата обновления |
| likes_count | int | Количество лайков (кэш) |

---

## 5. likes — Лайки

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| article_id | bigint, FK | Ссылка на articles |
| user_id | bigint, FK | Кто поставил |
| type | varchar | regular / expert |
| amount | int | Сумма (0 для regular, 100 для expert) |
| region_check | boolean | Прошёл ли геофильтр |
| affiliation_check | boolean | Прошёл ли аффилиационный фильтр |
| created_at | timestamp | Дата лайка |

**Ограничение:** 1 эксперт = 1 лайк на статью.

---

## 6. expert_cards — Экспертные карты

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| expert_id | bigint, FK | Ссылка на expert_profiles |
| balance | int | Текущий баланс (от 0 до 5000) |
| year | int | Год карты |
| issued_at | timestamp | Дата выпуска |
| expires_at | timestamp | Дата сгорания (31 декабря) |
| is_active | boolean | Активна ли |
| created_at | timestamp | Дата создания |

---

## 7. transactions — Транзакции

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| card_id | bigint, FK | Ссылка на expert_cards |
| author_id | bigint, FK | Ссылка на author_profiles |
| article_id | bigint, FK | Ссылка на articles |
| amount | int | Сумма (100) |
| status | varchar | pending / completed / cancelled |
| created_at | timestamp | Дата транзакции |

---

## 8. payouts — Выплаты авторам

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| author_id | bigint, FK | Ссылка на author_profiles |
| amount_gross | int | Начислено |
| tax | int | Налог (6%) |
| amount_net | int | К выплате |
| status | varchar | pending / processing / paid / failed |
| receipt_url | varchar | Ссылка на чек «Мой налог» |
| created_at | timestamp | Дата создания |
| paid_at | timestamp | Дата выплаты |

---

## 9. moderation_logs — Модерация

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| entity_type | varchar | user / article / like |
| entity_id | bigint | ID сущности |
| moderator_id | bigint, FK | Ссылка на users |
| action | varchar | approve / reject / block / warn |
| reason | text | Причина |
| created_at | timestamp | Дата |

---

## 10. roles — Роли (если нужно отдельно)

| Поле | Тип | Описание |
|---|---|---|
| id | bigint, PK | Идентификатор |
| name | varchar | reader / author / expert / moderator / admin |
| description | text | Описание |

Обычно роль — это просто поле в `users`.
Но если ролей станет много — можно вынести.

---

## Связи (ER-диаграмма, текстом)

users ──1:1── author_profiles
users ──1:1── expert_profiles
users ──1:N── likes
author_profiles ──1:N── articles
articles ──1:N── likes
expert_profiles ──1:1── expert_cards
expert_cards ──1:N── transactions
author_profiles ──1:N── payouts
users ──1:N── moderation_logs


---

## Индексы (для скорости)

- `users.email` — уникальный.
- `articles.status`, `articles.published_at`.
- `likes.article_id`, `likes.user_id`.
- `transactions.card_id`, `transactions.author_id`.
- `payouts.author_id`, `payouts.status`.

---

## Что важно помнить

- **1 эксперт = 1 лайк на статью.**
- **Геофильтр и аффилиация — проверяются.**
- **Налог 6% — удерживается автоматически.**
- **Баланс карты — 0…5000.**
- **Сгорание — 31 декабря.**

---

## Чего здесь нет

- ❌ Готовой SQL-схемы.
- ❌ Обязательных полей.
- ❌ Финальных решений.
- ❌ Гарантий, что это сработает без изменений.

Только набросок. Можно делать иначе. 🙂
