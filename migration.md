# Migration: lingomojo-landing → lingomojo-ext (mono-repo)

Перенос веб-сайта из отдельного публичного репо в единый приватный mono-repo.
После миграции сайт деплоится через Cloudflare Pages вместо GitHub Pages.

---

## Предварительно

- [ ] lingomojo-ext клонирован локально
- [ ] Доступ к Cloudflare Dashboard
- [ ] lingomojo-landing полностью запушен (все ветки актуальны)

---

## Шаг 1 — Клонировать lingomojo-ext (если ещё не сделано)

```bash
git clone git@github.com:pigeoncorner/lingomojo-ext.git d:\_git\lingomojo\ext
cd d:\_git\lingomojo\ext
```

---

## Шаг 2 — Импортировать web/ из lingomojo-landing

Выполнить в папке lingomojo-ext:

```bash
# Добавить landing как временный remote
git remote add landing https://github.com/pigeoncorner/lingomojo-landing.git
git fetch landing

# Переключиться на dev, импортировать web/ с историей
git checkout dev
git subtree add --prefix=web landing/dev --squash

# Переключиться на main, импортировать web/ с историей
git checkout main
git subtree add --prefix=web landing/main --squash

# Убрать временный remote
git remote remove landing
```

После этого в репо появится папка `web/` со всеми файлами сайта.

---

## Шаг 3 — Обновить .gitignore в lingomojo-ext

Добавить в корневой `.gitignore` репо:

```
# Web
web/.claude/
```

Остальное уже учтено в `web/.gitignore`.

---

## Шаг 4 — Запушить изменения

```bash
git checkout dev
git push origin dev

git checkout main
git push origin main
```

---

## Шаг 5 — Настроить Cloudflare Pages

1. Открыть **Cloudflare Dashboard → Workers & Pages → Create → Pages**
2. Нажать **Connect to Git** → выбрать репо `pigeoncorner/lingomojo-ext`
3. Заполнить настройки сборки:

   | Параметр | Значение |
   |---|---|
   | Production branch | `main` |
   | Build command | *(пусто)* |
   | Build output directory | `web` |

4. Нажать **Save and Deploy**
5. Дождаться первого деплоя (1-2 минуты)

---

## Шаг 6 — Подключить домен к Cloudflare Pages

1. В проекте Cloudflare Pages → **Custom domains → Add a custom domain**
2. Добавить `thelingomojo.com`
3. Добавить `www.thelingomojo.com`
4. Cloudflare автоматически создаст нужные DNS-записи

---

## Шаг 7 — Удалить старые DNS-записи GitHub Pages

После того как сайт открывается через Cloudflare Pages, удалить в **Cloudflare DNS** старые записи:

Удалить все A-записи `thelingomojo.com` с адресами `185.199.x.x`:
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

Удалить все AAAA-записи `thelingomojo.com` с адресами `2606:50c0:x::153`.

Удалить CNAME `www → pigeoncorner.github.io` (Cloudflare Pages добавит свою запись).

---

## Шаг 8 — Архивировать lingomojo-landing

1. Открыть https://github.com/pigeoncorner/lingomojo-landing
2. **Settings → Danger Zone → Archive this repository**
3. Подтвердить архивацию

---

## Результат

| | До | После |
|---|---|---|
| Web-репо | lingomojo-landing (публичный) | lingomojo-ext/web/ (приватный) |
| Хостинг | GitHub Pages | Cloudflare Pages |
| CDN | Fastly (GitHub) | Cloudflare |
| Деплой | push в lingomojo-landing | push в lingomojo-ext/web/ |
| CI/CD для server/ | без изменений | без изменений |

---

## Рабочий процесс после миграции

```
Изменения в web/   → git push → Cloudflare Pages автодеплой (1-2 мин)
Изменения в server/ → git push → GitHub Actions → VPS (без изменений)
```
