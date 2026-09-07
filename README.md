# Chebupelka

**Форк** [`alexey-goloburdin/chebupelka`](https://github.com/alexey-goloburdin/chebupelka) — оригінальний автор і задум належать йому, цей репозиторій лише продовжує й адаптує його роботу під власні потреби.

Мінімальний агент-помічник для програмування, який спілкується з LLM через OpenAI-сумісний API і виконує bash-команди.

Дивись YouTube-відео про розробку оригінального агента: [https://www.youtube.com/watch?v=H7FSTj4x4xQ](https://www.youtube.com/watch?v=H7FSTj4x4xQ).

## Можливості

- **Виклик інструментів** — модель викликає інструменти (за замовчуванням `bash`)
- **Цикл взаємодії** — агент може викликати інструменти, поки не вирішить задачу
- **OpenAI-сумісний API** — працює з Ollama, vLLM, LM Studio та будь-якими серверами з OpenAI-сумісним інтерфейсом
- **Без залежностей від SDK** — лише `requests`, жодних обгорток

## Швидкий старт

```bash
# Встановити залежності
uv sync

# Передати задачу аргументом
python chebupelka.py "Напиши скрипт на Python, який рекурсивно знайде всі .py файли в поточній директорії"
```

## Конфігурація

У `chebupelka.py` налаштуй підключення до LLM:

| Змінна          | Опис                                |
|-----------------|--------------------------------------|
| `LLM_BASE_URL`  | URL твого LLM-сервера                |
| `LLM_API_KEY`   | Ключ авторизації                     |
| `LLM_MODEL`     | Назва моделі                         |

### Приклад з Ollama

```python
LLM_BASE_URL = "http://127.0.0.1:11434/v1"
LLM_API_KEY = "ollama"  # або будь-який рядковий ключ
LLM_MODEL = "qwen3:1.7b"
```

## Як це працює

```
Користувач → Agent Loop → LLM (chat/completions API)
                       ↕
             Виклики інструментів (bash)
                       ↓
                 Результат команди
                       ↑
             Повернення в цикл з результатом
```

1. Користувач вводить задачу
2. Агент надсилає історію повідомлень у LLM
3. Якщо модель хоче викликати інструмент — результат виконується локально (`subprocess.run`)
4. Результат повертається моделі, цикл повторюється
5. Коли модель перестає викликати інструменти — задача завершена

## Розширення інструментів

Додай новий інструмент у `TOOLS_MAP`(тут — `call_tool`) і опиши його в `LLM_TOOLS`:

```python
# У LLM_TOOLS:
{
    "type": "function",
    "function": {
        "name": "read_file",
        "description": "Read the contents of a file.",
        "parameters": {
            "type": "object",
            "properties": {
                "path": {"type": "string", "description": "File path."},
            },
            "required": ["path"],
        },
    }
}

# У call_tool:
func = {"bash": run_bash, "read_file": read_file}.get(name)  # додати функцію
```

## Залежності

- Python ≥ 3.10
- `requests` — єдина runtime-залежність
