# Кэш-рынок: фоновая обработка заказов

Босс раздела PY11 школы BatSchool: заказы Барахолки идут в фон через outbox, очередь на Redis Streams и воркеров, которые помнят, что уже сделали. Условие заказа в школе, на странице этапа «Заказ: ночь, когда заказы не терялись». В репозитории только обвязка: зависимости, Redis и PostgreSQL в Docker, пример окружения и проверка в CI. Код твой: перенеси сюда каталог `ochered/` из своего решения десятого этапа (его видно в редакторе этапа на странице урока), закоммить и дальше дописывай заказ.

## Как работать локально

Нужны [uv](https://docs.astral.sh/uv/) и Docker.

```sh
uv sync                                   # Python 3.14 и зависимости
docker compose up -d                      # Redis 7 на :6379, PostgreSQL 17 на :5432
cp .env.example .env
docker compose exec redis redis-cli       # посмотреть ключи ochered:* руками
uv run --env-file .env celery -A ochered.celery_app worker --loglevel=INFO
uv run --env-file .env celery -A ochered.celery_app beat --loglevel=INFO
```

Свои тесты клади в `tests/` и запускай `uv run --env-file .env pytest`.

## Как проверяется

1. Привяжи свой репозиторий в школе: настройки, блок «Репозитории изделий», строка «Кэш-рынок: своя очередь и Celery на Redis», поле `владелец/репозиторий`, кнопка «Привязать». Владелец репозитория должен совпасть с твоим аккаунтом GitHub.
2. Каждый пуш запускает `.github/workflows/grade.yml`. Он вызывает общий workflow школы `bondiano/py-graders-ci`, тот поднимает Redis 7 и PostgreSQL 17, гоняет скрытые проверки образом `ghcr.io/bondiano/py-grader-ochered:11` и отправляет результат в школу. Перед каждой проверкой база Redis пустеет, а схема PostgreSQL строится заново. Зелёный прогон засчитывает этап.
3. Не меняй `grade.yml`: школа принимает отчёт только от общего workflow на опубликованном теге.

Тот же прогон можно сделать у себя. Грейдер дописывает свои файлы в каталог проекта, поэтому дай ему копию:

```sh
docker compose up -d
work="$(mktemp -d)" && cp -R . "$work" && rm -rf "$work/.git" && chmod -R a+rwX "$work"
docker run --rm --network host --read-only --tmpfs /tmp \
  -e DATABASE_URL=postgresql://postgres:postgres@localhost:5432/postgres \
  -e REDIS_URL=redis://localhost:6379/0 \
  -v "$work:/work" ghcr.io/bondiano/py-grader-ochered:11
```

Каждая строка вывода это одна проверка: `"passed": true` или `false` и сообщение.
