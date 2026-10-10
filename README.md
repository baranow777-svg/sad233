# Сад семьи Барановых — облачный атлас и дневник растений

**Самара, микрорайон Орловский, 233**

Интерактивный атлас и дневник растений сада: карточки растений, дневник (дата посадки, возраст, записи, фотографии), архив растений, поиск, общий дневник, импорт/экспорт в Excel.

Эта версия сайта **синхронизирует изменения между всеми устройствами** через **Supabase** (PostgreSQL + Realtime + Storage в одном сервисе). Данные, добавленные на телефоне, автоматически появляются на компьютере и наоборот.

> 💡 Почему Supabase? Это открытый бесплатный сервис (не Google), где в одном месте и база данных, и хранилище файлов, и мгновенная синхронизация.

## Возможности

- ✅ **Автосинхронизация**: карточки, дневники, архив — общие на всех устройствах.
- ✅ **Фото в облаке**: загружаются в Supabase Storage, видны на всех устройствах по ссылке.
- ✅ **Бесплатный тариф** Supabase достаточен для нескольких пользователей/устройств.

## Как разместить на GitHub Pages

1. Создайте репозиторий на GitHub.
2. Залейте `index.html` и `README.md`.
3. Репозиторий → **Settings → Pages** → *Deploy from a branch* → ветка `main`, папка `/ (root)`.
4. Сайт: `https://<ваш-логин>.github.io/<имя-репо>/`.

> ⚠️ Синхронизация работает только при открытии сайта по **https** (GitHub Pages даёт их автоматически).

## Настройка Supabase (однократно, ~20 мин)

### 1. Создать проект
1. [supabase.com](https://supabase.com) → **Start your project** → войти (GitHub/почта).
2. **New project**: Name `sad-baranovyh`, свой password БД, регион `eu-central-1`, план **Free** → Create.

### 2. Вставить URL и ключ в index.html
1. Меню **Project Settings → API**.
2. Скопируйте **Project URL** (`https://xxxx.supabase.co`) и **anon/public key**.
3. Откройте `index.html`, найдите блок:
```html
<script>
  var supabaseConfig = {
    url: "https://XXXXXXXX.supabase.co",
    anonKey: "eyJhbGciOi..."
  };
```
Замените `https://XXXXXXXX.supabase.co` на ваш Project URL, а `eyJhbGciOi...` — на ваш anon-ключ.

> ⚠️ **anon key** можно класть в сайт. Секретный **service_role key** класть нельзя.

### 3. Создать таблицу `garden_state`
1. **Table Editor → New table**.
2. Name: `garden_state`.
3. Строка `id` → тип `text`, primary key, **default `'main'`**.
4. Столбец `data` → тип `jsonb`.
5. Save.

### 4. Включить Realtime для таблицы
**Database → Replication → Realtime** → включить **`garden_state`**.

### 5. Создать bucket `photos`
**Storage → New bucket** → name `photos`, **public: включить**.

### 6. Безопасность (RLS) — важно!
**SQL Editor → New query → Run**:
```sql
alter table "public"."garden_state" enable row level security;

create policy "anon read" on "public"."garden_state"
  for select using (true);

create policy "anon insert" on "public"."garden_state"
  for insert with check (true);

create policy "anon update" on "public"."garden_state"
  for update using (true) with check (true);
```

### 7. Storage политики
**Storage → Policies → New policy** для bucket `photos`, allow read/write for anon:
```sql
create policy "photos public read" on storage.objects
  as permissive for select using (bucket_id = 'photos');

create policy "photos anon upload" on storage.objects
  as permissive for insert with check (bucket_id = 'photos');
```

## Структура данных
- **Таблица** `garden_state` (строка `id='main'`): JSON в столбце `data` → `{overrides, diaries, archive}`.
- **Realtime** передаёт изменения между открытыми устройствами.
- **Storage**: фотографии в bucket `photos`.

## Перенос существующих данных
Если вы уже вносили данные в локальную версию — откройте её и через **Экспорт (Excel)**, затем здесь — **Импорт (Excel)**.

## Авторство
Сад семьи Барановых · Самара, микрорайон Орловский, 233