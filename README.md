        # redis — Redis Cluster — шардинг

        Homework-шаблон для урока **l3_cluster** (Redis Cluster — шардинг) на платформе Vibe Learn.

        ## Что делать

        docker-compose: 3 мастера + 3 реплики. Go-клиент через redis.NewClusterClient. Скрипт:
нагружает разные слоты, измеряет распределение, демонстрирует MOVED-redirect, показывает
multi-key с hash tag работает а без — CROSSSLOT error. Тесты проверят: hash-slot
распределение равномерное, failover мастера через kill -9 работает.

## Контекст (из transfer-задачи урока)

Сайт e-commerce. Сейчас Redis Sentinel + 1 мастер на 96GB RAM, 90% утилизация.
Команда планирует переход на Cluster. Профиль данных: cart:user_id:items (hash),
sessions:session_id (string), product:id:info (hash). Опиши схему hash-tags чтобы
multi-key операции (вроде «корзина + рекомендации для юзера») продолжали работать,
и обсуди trade-off.

## Recap из урока

- Cluster = 16384 hash slot'a, распределённых по мастерам. CRC16(key) % 16384 = slot.
- Multi-key операции (MULTI, Lua, SINTERSTORE...) работают ТОЛЬКО если все ключи в одном slot. Иначе CROSSSLOT error.
- Hash tag в `{...}` — подстрока, по которой считается slot. Группируй связанные ключи юзера: `user:{42}:profile`, `user:{42}:cart`.
- Минимум 6 нод (3 master + 3 replica) для production. Failover автоматический через gossip — Sentinel не нужен.
- Cluster нужен когда: keyspace > RAM, write > 100k RPS, естественное разбиение по tenant'ам. Иначе Sentinel проще.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - Docker + docker-compose — `docker compose up -d` поднимает single-node Redis 7 на `localhost:6379` (с включёнными keyspace-notifications и AOF). Адрес переопределяется через env `REDIS_ADDR`.

        ## Запуск

        ```bash
        # Поднять локальный Redis
        docker compose up -d

        # Прогнать тесты (интеграционный включается через REDIS_INTEGRATION=1)
        go test ./...
        REDIS_INTEGRATION=1 go test ./...

        # Запустить main (печатает marker; замени stub на реализацию)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
