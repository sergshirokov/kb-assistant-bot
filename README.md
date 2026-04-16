# 🤖 KB Assistant Bot

Telegram-бот с **RAG** (Retrieval-Augmented Generation): отвечает на вопросы, опираясь на ваши документы в векторной базе **ChromaDB**. Поиск по смыслу, ответ формирует LLM — **OpenAI** или **GigaChat** (выбор через переменные окружения).

> 💡 Для **эмбеддингов** (индексация и поиск в ChromaDB) в любом режиме используется **OpenAI API** — ключ `OPENAI_API_KEY` обязателен даже при `AI_PROVIDER=gigachat`.

---

## 📋 Требования

- **Python 3.10+**
- Аккаунт и токен **Telegram Bot** ([@BotFather](https://t.me/BotFather))
- Ключ **OpenAI** (чат и/или эмбеддинги — см. ниже)
- При выборе GigaChat — ключ **GigaChat** и пакет **`requests`** (для HTTP-клиента; при необходимости: `pip install requests`)

---

## ⚙️ Установка

Из **корня репозитория**:

```bash
python -m venv .venv
```

**Windows (PowerShell):**

```bash
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

**Linux / macOS:**

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

---

## 🔐 Переменные окружения

Создайте файл **`.env`** в корне проекта (или экспортируйте переменные в системе).

### Обязательные

| Переменная | Описание |
|------------|----------|
| `TELEGRAM_BOT_TOKEN` | Токен Telegram-бота |
| `OPENAI_API_KEY` | Ключ OpenAI (эмбеддинги; при `AI_PROVIDER=openai` — ещё и генерация ответов) |

### Провайдер ответов

| Переменная | Описание |
|------------|----------|
| `AI_PROVIDER` | `openai` (по умолчанию) или `gigachat` |

Если `AI_PROVIDER=gigachat`, дополнительно:

| Переменная | Описание |
|------------|----------|
| `GIGACHAT_AUTHORIZATION_KEY` | Ключ авторизации GigaChat |

### Необязательные

| Переменная | По умолчанию | Описание |
|------------|---------------|----------|
| `OPENAI_MODEL` | `gpt-4o-mini` | Модель чата OpenAI |
| `OPENAI_EMBEDDING_MODEL` | `text-embedding-3-small` | Модель эмбеддингов |
| `GIGACHAT_MODEL` | `GigaChat` | Модель GigaChat |
| `GIGACHAT_TEMPERATURE` | `0.7` | Температура GigaChat |
| `GIGACHAT_MAX_TOKENS` | `1000` | Лимит токенов ответа GigaChat |
| `CHROMA_PERSIST_DIR` | `./chroma_db` | Каталог ChromaDB |
| `CHROMA_COLLECTION` | `documents` | Имя коллекции |

---

## 📚 База знаний (индексация)

1. Положите исходные файлы в каталог **`data/`** (форматы: **`.txt`**, **`.html`**, **`.htm`**).
2. Выполните из **корня проекта** (с активированным venv и настроенным `.env`):

```bash
python -m tools.ingest_documents
```

Опции:

```bash
python -m tools.ingest_documents --files path/to/a.txt path/to/b.html
python -m tools.ingest_documents --chunk-size 500 --overlap 100
```

После успешного прогона появится (или обновится) каталог **`chroma_db/`** с проиндексированными фрагментами документов.

> ⚠️ Скрипт индексации загружает настройки через `Settings.from_env()` — нужны те же обязательные переменные, что и для бота (включая `TELEGRAM_BOT_TOKEN` и ключи по выбранному `AI_PROVIDER`).

---

## 🚀 Запуск

### 💻 Универсальный способ (рекомендуется)

Из **корня репозитория**, с активированным виртуальным окружением:

```bash
python main.py
```

Альтернатива:

```bash
python run.py
```

### 🪟 Windows

Те же команды в **PowerShell** или **cmd** из каталога проекта (после `.\.venv\Scripts\activate` и `pip install -r requirements.txt`). Отдельные `.bat`-скрипты в репозитории не используются.

Перед первым запуском убедитесь, что каталог **`chroma_db/`** уже создан (см. раздел про индексацию).

---

## 💬 Использование

1. Найдите вашего бота в Telegram.
2. Отправьте **`/start`**.
3. Задайте вопрос обычным текстом.
4. Получите ответ на основе базы знаний.

### Команды бота

- **`/start`** — приветствие и краткая справка
- **`/help`** — подробная справка
- **`/stats`** — статистика базы знаний, пользователей и активных сессий
- **`/clear`** — очистить историю диалога

---

## 📁 Структура проекта

```
.
├── main.py                      # Точка входа
├── run.py                       # Запуск через обёртку
├── requirements.txt             # Зависимости Python
├── README.md                    # Документация
├── .env                         # Локальные секреты (не коммитить)
├── user_data.json               # Данные пользователей бота (создаётся при работе)
├── data/                        # Исходные документы для индексации
├── chroma_db/                   # Персистентная ChromaDB (после ingest)
│
├── config/                      # Конфигурация
│   ├── __init__.py
│   └── settings.py              # Настройки из окружения
│
├── interface/                   # Интерфейс: Telegram
│   ├── __init__.py
│   ├── telegram_bot.py
│   └── handlers.py
│
├── dialog_controller/           # Сессии и контекст диалога
│   ├── __init__.py
│   ├── session_manager.py
│   └── user_context.py
│
├── memory_manager/              # Промпты и RAG-контекст
│   ├── __init__.py
│   ├── prompt_builder.py
│   └── context_retriever.py
│
├── ai_processor/                # OpenAI: клиент и генерация ответов
│   ├── __init__.py
│   ├── openai_client.py
│   └── response_generator.py
│
├── ai_gigachat_processor/       # GigaChat: клиент и генерация ответов
│   ├── __init__.py
│   ├── config.py
│   ├── gigachat_client.py
│   └── response_generator.py
│
├── storage/                     # Векторная БД и пользователи
│   ├── __init__.py
│   ├── vector_db.py
│   ├── user_db.py
│   └── document_loader.py
│
├── tools/                       # Утилиты
│   ├── __init__.py
│   └── ingest_documents.py      # Загрузка документов в ChromaDB
│
└── utils/                       # Общие утилиты
    ├── __init__.py
    └── logging_config.py
```

---

## ✅ Преимущества модульной архитектуры

- ✅ **Разделение ответственности** — каждый модуль решает свою задачу  
- ✅ **Удобное тестирование** — модули слабо связаны  
- ✅ **Масштабируемость** — проще добавлять функции и провайдеры  
- ✅ **Переиспользование** — компоненты можно вызывать отдельно  
- ✅ **Читаемость** — предсказуемая структура каталогов  

---

## 🔧 Расширение функциональности

### Новый интерфейс (API, Web, CLI)

Подключайте существующие компоненты из корня проекта (запуск из корня, `PYTHONPATH` = корень):

```python
from dialog_controller import SessionManager
from memory_manager import ContextRetriever
from ai_processor import ResponseGenerator
```

### Другое хранилище

Добавьте реализацию в **`storage/`** и подключите её в **`main.py`** вместо текущих классов.

### Другая модель или провайдер

- Параметры OpenAI / GigaChat — через **`.env`** (см. таблицы выше).  
- Новый провайдер — отдельный пакет по аналогии с **`ai_gigachat_processor/`** и ветка в **`initialize_components()`** в **`main.py`**.

---

## 📜 Логирование

Логи выводятся в **консоль**. Уровень задаётся при старте в **`main.py`**:

```python
setup_logging(level="DEBUG")  # DEBUG, INFO, WARNING, ERROR
```

---

## 📄 Лицензия

Свободно используйте для обучения и коммерческих проектов.
