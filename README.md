# DocuForge AI

`DocuForge AI` — веб-сервис на `FastAPI` для автоматической генерации Markdown-документации по исходному коду репозитория или ZIP-архива.

Проект анализирует структуру кода, группирует файлы по зависимостям, передаёт контекст в LLM и сохраняет результат в виде документации. Это портфолио-пример backend-сервиса с асинхронной обработкой задач, GitHub-интеграцией, Redis-уведомлениями и генерацией технических документов.

## Что делает проект

- принимает ссылку на GitHub-репозиторий;
- клонирует репозиторий во временную директорию;
- принимает ZIP-архив с кодом;
- анализирует зависимости между файлами;
- группирует связанные файлы;
- генерирует Markdown-документацию для групп файлов;
- сохраняет результат в storage;
- записывает информацию о репозитории в базу данных;
- уведомляет пользователя о готовности через Redis Pub/Sub / WebSocket-сценарий.

## Стек

| Зона | Технологии |
|---|---|
| Backend | `Python`, `FastAPI`, `Uvicorn` |
| Templates / Static | `Jinja2`, `StaticFiles` |
| Database | `SQLAlchemy`, async session, PostgreSQL-compatible stack |
| Queue / PubSub | `Redis` |
| AI integration | `OpenAI-compatible client`, DeepSeek-compatible endpoint |
| Parsing | custom dependency analyzer for Python, JS, TS, HTML, CSS, Vue, Rust and other file types |
| Infrastructure | `.env`, local storage, background tasks |

## Основные возможности

### Repository processing

Сервис принимает URL репозитория, получает информацию о нём, клонирует код и запускает фоновую обработку.

```text
repo_url -> GitHubService -> clone -> DependencyAnalyzer -> documentation generation -> storage -> DB
```

### ZIP processing

Можно загрузить ZIP-архив с кодом. Сервис распаковывает архив во временную директорию и запускает тот же pipeline генерации документации.

### Dependency analysis

`DependencyAnalyzer` собирает файлы по поддерживаемым расширениям и строит граф зависимостей. Поддерживаются разные группы файлов:

- Python;
- JavaScript / TypeScript;
- React JSX / TSX;
- HTML / CSS / SCSS;
- Vue / Svelte;
- Rust;
- Go;
- Java;
- C / C++ / C#;
- YAML / JSON / TOML;
- SQL;
- другие распространённые форматы.

### AI documentation generation

`AIService` разбивает большие файлы на чанки, строит prompt для технической документации и вызывает LLM через OpenAI-compatible client.

Документация генерируется на русском языке и включает:

- описание классов;
- описание методов и функций;
- входные параметры;
- ожидаемые результаты;
- зависимости файла;
- Markdown-структуру;
- технические замечания.

## Архитектура проекта

```text
Generator-Documentation/
├── src/
│   ├── api/
│   │   ├── main_routes.py          # основные страницы
│   │   ├── auth_routes.py          # авторизация
│   │   ├── github_routes.py        # GitHub-интеграции
│   │   ├── repo_routes.py          # обработка репозиториев
│   │   ├── docs_routes.py          # доступ к документации
│   │   └── profile_routes.py       # профиль пользователя
│   ├── services/
│   │   ├── repo_service.py         # pipeline обработки репозитория
│   │   ├── github_service.py       # GitHub operations
│   │   ├── ai_service.py           # LLM-интеграция
│   │   ├── auth_service.py         # auth business logic
│   │   └── auth_handler.py         # auth helpers
│   ├── utils/
│   │   ├── dependency_analyzer.py  # анализ зависимостей
│   │   └── doc_generator.py        # генерация Markdown-документов
│   ├── models/                     # модели данных
│   ├── schemas/                    # Pydantic-схемы
│   ├── redis.py                    # Redis service
│   └── main.py                     # создание FastAPI app
├── static/
├── storage/
├── requirements.txt
└── README.md
```

## Основные маршруты

| Prefix | Назначение |
|---|---|
| `/` | основные страницы |
| `/auth` | авторизация |
| `/github` | GitHub-интеграции |
| `/repos` | обработка репозиториев и ZIP-архивов |
| `/docs` | просмотр / получение документации |
| `/profile` | профиль пользователя |

## Pipeline обработки

```text
1. Пользователь передаёт repo URL или ZIP.
2. Backend создаёт repo_id.
3. Код клонируется или распаковывается во временную директорию.
4. DependencyAnalyzer собирает файлы и строит группы зависимостей.
5. Для каждой группы запускается генерация документации.
6. Markdown-файлы сохраняются в storage/docs/<user_id>/<repo_id>.
7. Метаданные сохраняются в БД.
8. Пользователь получает уведомление о готовности.
```

## Быстрый старт

### 1. Клонировать репозиторий

```bash
git clone https://github.com/6oT9lpa/Generator-Documentation.git
cd Generator-Documentation
```

### 2. Создать виртуальное окружение

```bash
python -m venv .venv
source .venv/bin/activate
```

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 3. Установить зависимости

```bash
pip install -r requirements.txt
```

### 4. Подготовить `.env`

Пример переменных:

```env
TOKEN_OPENAI=<llm_api_token>
DATABASE_URL=<database_url>
REDIS_URL=<redis_url>
SECRET_KEY=<secret_key>
```

Названия переменных могут отличаться в зависимости от текущей реализации `config.py`, поэтому перед запуском проверьте файл конфигурации проекта.

### 5. Запустить приложение

```bash
uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload
```

После запуска:

```text
http://localhost:8000
```

## Технические особенности

- Фоновая обработка запускается через `asyncio.create_task`.
- Генерация документации для групп файлов выполняется через `ProcessPoolExecutor`.
- Результаты сохраняются в файловую систему.
- Redis используется для уведомления пользователя о готовности документации.
- Анализатор зависимостей поддерживает несколько языков и форматов файлов.
- LLM-вызовы выполняются через OpenAI-compatible клиент.

## Что проект показывает работодателю

- Умение строить backend-сервис на FastAPI.
- Асинхронная обработка задач.
- Интеграция с внешним API / LLM API.
- Работа с GitHub-репозиториями и ZIP-архивами.
- Проектирование pipeline: input -> processing -> storage -> notification.
- Работа с Redis Pub/Sub.
- Написание собственного анализатора зависимостей.
- Генерация технической документации в Markdown.

## Статус проекта

Пет-проект / хакатонный портфолио-проект. Основной фокус — backend pipeline, анализ кода, интеграция с LLM и генерация документации.
