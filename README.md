# drupal-sitemap-check

Одностраничный инструмент для Drupal: проверяет, есть ли URL в `sitemap.xml`, и определяет, какая сущность за ним стоит — тип, id и прямую ссылку на сущность.
&nbsp;&nbsp;|&nbsp;&nbsp;A single-file tool for Drupal: checks whether a URL exists in `sitemap.xml` and resolves which entity it belongs to — type, id and a direct entity link.

**Язык / Language:** [Русский](#ru) · [English](#en) — интерфейс инструмента на русском / the tool UI is Russian.

<a id="ru"></a>

<details open>
<summary><b>🇷🇺 Русский</b></summary>

## Что это

`drupal-sitemap-check.html` — один файл, без зависимостей, без сборки, без запросов к сторонним сервисам. Всё считается в браузере.

Он закрывает два вопроса:

1. **Есть ли этот URL в карте сайта?** — для списка URL (из отчёта, из Search Console, из Топвизора, из таблицы) или для всех URL самой карты.
2. **Что за страница?** — тип сущности (`node`, `taxonomy_term`, `user`, `media`, `view`, `node_type`), её id и ссылка вида `/node/135`, `/taxonomy/term/11`.

Типичный сценарий: в карте сайта лежат URL, которые отдают 301 на другие страницы. Их надо найти и убрать.

## Как запустить

Браузер блокирует `fetch()` на чужой домен, поэтому файл нужно открывать **с того же домена, который проверяете**:

```bash
# положить файл в корень сайта
cp drupal-sitemap-check.html /var/www/drupal/
# или в корень фронтенда
cp drupal-sitemap-check.html /var/www/html/
```

```
https://ваш-сайт/drupal-sitemap-check.html
```

Если Drupal лежит в поддиректории (`https://site.ru/drupal/`), положите файл рядом с `index.php` — база (`BASE`) возьмётся из адреса страницы автоматически.

Открыть файл как `file://` можно, но `sitemap.xml` и страницы не загрузятся (CORS). Как обходной путь: список URL можно вставить текстом или загрузить файлом, а sitemap отдаётся вставкой XML.

## Как пользоваться

1. **Sitemap** — оставьте пустым: инструмент сам возьмёт адрес из `robots.txt`, а если там нет строки `Sitemap:` — попробует `/sitemap.xml`. Можно вставить XML карты прямо в поле или указать свой URL (в том числе `sitemap index` — дети подтянутся автоматически).
2. **Список URL** — по одному на строке, можно относительными путями (`/page`), полными URL или просто списком из файла (`.txt`, `.csv`). **Пустое поле = проверяются сами URL из `sitemap.xml`.**
3. **Проверить** — и смотрите таблицу.

Опции:

| Опция | Что делает |
|---|---|
| определять сущность | скачивает каждую страницу и заполняет колонки сущности (id, тип, view, бандл, title, редирект) |
| редирект считать найденным по целевой | если самого URL в карте нет, но его финальная страница есть — метка «только цель», а не «нет» |
| максимум URL | ограничение количества проверяемых URL (по умолчанию 100) |

## Что в таблице

| Колонка | Значение |
|---|---|
| URL | путь, HTTP-статус |
| В sitemap.xml | `да` / `нет` / `только цель` |
| Тип сущности | `node`, `taxonomy_term`, `user`, `media`, `view`, `route` |
| ID | id сущности |
| Внутренний путь | `drupalSettings.path.currentPath` — например `node/135` |
| Ссылка на сущность | кликабельная `/node/135`, `/taxonomy/term/11` |
| Редирект | конечный URL при 301/302 или `meta refresh` |
| Бандл / словарь | из классов `node--type-*` / `taxonomy-term--vocabulary-*` |
| View | имя views из `data-drupal-view-name` |
| Title | `<title>` страницы |
| Флаги | 404, 403, meta refresh, JS-защита, canonical → другой URL |

По умолчанию сортировка такая, что **«нет в карте» сверху**. Клик по заголовку — сортировка по колонке. Кнопки-фильтры: `все`, `нет в карте`, `редиректы`, `404 / ошибки`, `есть в карте`. Есть экспорт в CSV (разделитель `;`, UTF-8 BOM — открывается в Excel).

Внизу — блок «Разбор одной ссылки»: вставьте один URL и получите полный отчёт по нему.

## Как определяется id сущности

Главный источник — **`drupalSettings.path.currentPath`** в HTML страницы: внутренний путь Drupal (`node/135`, `taxonomy/term/11`, `user/1`, `view/frontpage`). Он работает всегда, в том числе когда:

- тема не добавляет классы `<body>` (а многие кастомные темы их не добавляют);
- `canonical` переписан SEO-модулем на сам запрошенный URL;
- страница отдаётся View, а не отдельной нодой.

Резервные источники: `canonical` вида `/node/N`, `data-history-node-id` у views-контейнера, сам URL.

Если страница отдаёт `200` + `meta refresh` и не содержит Drupal-данных — это, как правило, антибот/челлендж; такой флаг помечается отдельно, id по нему недоступен.

## Что делать с результатами

Найденные в карте URL с редиректом — это почти всегда старые алиасы тех же самых сущностей (их актуальные адреса в карте тоже есть). Правильное лечение — убрать старые алиасы, а не исключать пути:

- `Администрирование → Контент → URL-адреса` (`/admin/config/search/path`) — найти алиас и его «исходный путь», снять с публикации или удалить;
- служебные пути вида `/node/1`, `/taxonomy/term/11` в карте — у сущности нет алиаса, поэтому она попала в карту «как есть»;
- после правок пересобрать карту: `drush sm::rebuild` или кнопка на `/admin/config/search/sitemap`.

Проверить, что в карте больше нет мусора: оставьте список URL пустым, нажмите «Проверить», отсортируйте по «редиректы» и «404».

## Ограничения

- Нужен доступ к сайту **с того же домена** (см. «Как запустить»), иначе браузер заблокирует запросы.
- Работает с Drupal, который отдаёт `drupalSettings` в разметке (это Drupal 8+).
- Инструмент не авторизуется: видит ровно то, что видит анонимный посетитель. Закрытые страницы придут как 403/редирект на логин.
- Для sitemap index читаются первые 60 дочерних файлов, для проверки страниц — лимит из поля «максимум URL».
- Защита от паразитов (antibot, капча) может отдавать вместо страницы challenge — такие строки помечаются флагом и не дают id.

## Приватность

Всё локально: запросы идут только к тому домену, откуда открыт файл, данные никуда не отправляются, внешних скриптов и аналитики нет. Cookie отправляются только same-origin и только того домена, где открыт инструмент.

Интерфейс — русский, английская версия только в этом README.

</details>

<a id="en"></a>

<details open>
<summary><b>🇬🇧 English</b></summary>

## What it is

`drupal-sitemap-check.html` is a single file, no dependencies, no build step, no third-party requests. Everything is computed in the browser.

It answers two questions:

1. **Is this URL present in the sitemap?** — for a list of URLs (a report, Search Console, an SEO audit, a spreadsheet) or for every URL in the sitemap itself.
2. **What is this page?** — the entity type (`node`, `taxonomy_term`, `user`, `media`, `view`, `node_type`), its id, and a link such as `/node/135`.

Typical case: the sitemap contains URLs that 301-redirect elsewhere. You need to find and remove them.

## How to run it

Browsers block cross-origin `fetch()`, so the file must be opened **from the same domain you are checking**:

```bash
# put the file into the Drupal web root
cp drupal-sitemap-check.html /var/www/drupal/
# or into the front controller document root
cp drupal-sitemap-check.html /var/www/html/
```

```
https://your-site.tld/drupal-sitemap-check.html
```

If Drupal lives in a subdirectory (`https://site.tld/drupal/`), drop the file next to `index.php` — the base URL (`BASE`) is derived from the page address automatically.

Opening the file as `file://` works, but `sitemap.xml` and pages will not load (CORS). Workaround: paste the URL list or upload a file, and paste the sitemap XML directly into the field.

## Usage

1. **Sitemap** — leave empty and the tool takes the URL from `robots.txt`, falling back to `/sitemap.xml`. You can also paste the sitemap XML into the field or type any URL (a `sitemap index` is expanded automatically).
2. **URL list** — one per line: relative paths (`/page`), absolute URLs, or a list loaded from a `.txt` / `.csv` file. **Leave it empty to check the URLs of `sitemap.xml` itself.**
3. **Check** — read the table.

Options:

| Option | Effect |
|---|---|
| resolve entity | fetches each page and fills entity columns (id, type, view, bundle, title, redirect) |
| count redirect target as found | if the URL itself is not in the sitemap but its final page is, the label is "target only" instead of "no" |
| max URLs | limit of checked URLs (default 100) |

## Table columns

| Column | Meaning |
|---|---|
| URL | path, HTTP status |
| In sitemap.xml | `yes` / `no` / `target only` |
| Entity type | `node`, `taxonomy_term`, `user`, `media`, `view`, `route` |
| ID | entity id |
| Internal path | `drupalSettings.path.currentPath`, e.g. `node/135` |
| Entity link | clickable `/node/135`, `/taxonomy/term/11` |
| Redirect | final URL of a 301/302 or `meta refresh` |
| Bundle / vocabulary | from `node--type-*` / `taxonomy-term--vocabulary-*` classes |
| View | views name from `data-drupal-view-name` |
| Title | page `<title>` |
| Flags | 404, 403, meta refresh, bot protection, canonical → other URL |

Default sorting puts **"no" (missing from the sitemap) on top**. Click a header to sort by that column. Filter buttons: `all`, `no`, `redirects`, `404 / errors`, `yes`. CSV export uses `;` as the separator with a UTF-8 BOM, so it opens in Excel.

At the bottom there is a single-link inspector: paste one URL and get the full report for it.

## How the entity id is resolved

The primary source is **`drupalSettings.path.currentPath`** in the page HTML: Drupal's internal path (`node/135`, `taxonomy/term/11`, `user/1`, `view/frontpage`). It always works, including when:

- the theme adds no `<body>` classes (many custom themes do not);
- `canonical` has been rewritten by an SEO module to the requested URL;
- the page is rendered by a View rather than a single node.

Fallbacks: a `canonical` containing `/node/N`, `data-history-node-id` on the views container, and the URL itself.

If a page returns `200` + `meta refresh` with no Drupal data, that is usually a bot-protection challenge. Such rows are flagged and expose no id.

## What to do with the results

Redirecting URLs inside the sitemap are almost always stale path aliases of the very same entities (their current URLs are in the sitemap too). The correct fix is to remove the stale aliases, not to exclude paths:

- `Administration → Content → URL aliases` (`/admin/config/search/path`) — find the alias and its source path, unpublish or delete it;
- service URLs such as `/node/1` or `/taxonomy/term/11` appear because the entity has no alias, so it landed in the sitemap as-is;
- rebuild the sitemap afterwards: `drush sm::rebuild`, or the button at `/admin/config/search/sitemap`.

To verify the sitemap is clean: leave the URL list empty, press Check, then filter by `redirects` and `404 / errors`.

## Limitations

- The tool needs to be served from the same domain (see "How to run it"), otherwise the browser blocks the requests.
- Works with Drupal versions that expose `drupalSettings` in the markup (Drupal 8+).
- It does not authenticate: it sees exactly what an anonymous visitor sees. Protected pages come back as 403 or a redirect to the login form.
- For a sitemap index only the first 60 child files are read; page checks are limited by the "max URLs" field.
- Bot protection (antibot, captcha) may return a challenge instead of the page — such rows are flagged and yield no id.

## Privacy

Everything is local: requests go only to the domain the file was opened from, nothing is uploaded anywhere, there are no external scripts and no analytics. Cookies are sent same-origin only, to the same domain as the tool.

The UI is Russian; this README is bilingual.

</details>

## Credits / Автор

Author: [@Mmitekk](https://github.com/Mmitekk)
