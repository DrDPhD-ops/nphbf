# Egress check — Claude Code remote session (2026-09-21)

## Итог одной строкой

**Переносить работу сюда нет смысла.** Этот контейнер сидит за тем же
policy-enforcing egress-прокси, что и родительская сессия. Все девять доменов из
проверочного списка блокируются на уровне CONNECT (гейтвей отвечает `403`, curl
показывает `000`). `WebFetch` блокируется точно так же — ошибка
`EGRESS_BLOCKED`. Единственное, что реально открыто наружу: **GitHub**
(`raw.githubusercontent.com`, `api.github.com`), **PyPI** и инструмент
**`WebSearch`** (он ходит не через этот прокси).

## Задача 1 — результат требуемой команды

Команда:

```
for u in ...; do echo "$(curl -s -o /dev/null -m 25 -w '%{http_code}' -A 'Mozilla/5.0' "$u")  $u"; done
```

| HTTP | URL | Вердикт |
|------|-----|---------|
| `000` | https://ashyxchen.me | блок |
| `000` | https://larissaterranova.com | блок |
| `000` | https://swandaru.framer.ai | блок |
| `000` | https://www.researchgate.net/profile/Jackson-Trager | блок |
| `000` | https://www.linkedin.com/in/ashyxchen | блок |
| `000` | https://api.crossref.org/works?rows=1 | блок |
| `000` | https://pub.orcid.org/v3.0/expanded-search/?q=family-name:Leidinger | блок |
| `000` | https://api.openalex.org/authors?search=test | блок |
| `000` | https://dblp.org/search/author/api?q=test&format=json | блок |

**200 не ответил ни один домен из списка.**

`000` здесь — это не таймаут и не DNS. Прокси отказывает в туннеле:

```
> CONNECT api.crossref.org:443 HTTP/1.1
< HTTP/1.1 403 Forbidden
* CONNECT tunnel failed, response 403
```

`curl -sS "$HTTPS_PROXY/__agentproxy/status"` подтверждает по каждому из девяти
хостов:

```
"kind": "connect_rejected",
"detail": "gateway answered 403 to CONNECT (policy denial or upstream failure)"
```

Это организационная egress-политика, а не поломка. README прокси
(`/root/.ccr/README.md`) прямо запрещает ретраить и обходить `403/407` — их
надо только репортить, что и сделано.

## Дополнительная разведка: что здесь всё-таки открыто

| HTTP | URL | Вердикт |
|------|-----|---------|
| `200` | https://raw.githubusercontent.com/torvalds/linux/master/README | **открыт** |
| `200` | https://api.github.com | **открыт** |
| `200` | https://pypi.org/simple/ | **открыт** |
| `400` | https://github.com | **открыт** (400 — это git-прокси на голом GET, хост доступен) |
| `000` | https://arxiv.org/abs/1706.03762 | блок |
| `000` | https://arxiv.org/html/2404.01268v1 | блок |
| `000` | https://aclanthology.org | блок |
| `000` | https://openreview.net | блок |
| `000` | https://orcid.org | блок |
| `000` | https://scholar.google.com | блок |
| `000` | https://api.semanticscholar.org/graph/v1/paper/search | блок |
| `000` | https://www.google.com | блок |

## Что это значит для Задачи 2

Из пяти источников в списке приоритетов живым остаётся ровно один:

1. ~~Личный сайт / портфолио~~ — фетч невозможен (`curl` и `WebFetch` оба блок).
   `curl -s <url> | grep -oE 'mailto:[^"]+'` здесь не выполним в принципе.
2. **GitHub: профиль, README, `pyproject.toml`, `setup.py`, `.cff`** — работает
   полностью (`api.github.com` + `raw.githubusercontent.com` + MCP-инструменты).
   Плюс адреса авторов коммитов через `/repos/{o}/{r}/commits`.
3. ~~Страница лаборатории / вузовский справочник~~ — фетч невозможен.
4. ~~ORCID, OpenReview, ACL Anthology, DBLP~~ — все четыре блокируются.
5. ~~Полный текст arXiv (`arxiv.org/html/<id>`)~~ — блокируется.

`WebSearch` работает и лимит на него есть, но он возвращает заголовки, URL и
сниппеты — открыть найденную страницу и прочитать напечатанный на ней адрес
здесь нельзя. Поэтому поиск годится только чтобы выйти на GitHub-хэндл, который
дальше уже проверяется по API.

Вывод: сбор почт с личных сайтов в этом окружении невыполним. Выполнимая часть —
только GitHub-ветка, и она покрывает малую долю списка (большинство строк — это
LinkedIn-профили людей без публичного GitHub).
