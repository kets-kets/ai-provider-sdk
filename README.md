# 🤖 AI Provider SDK

> Унифицированный SDK для работы с AI-провайдерами (Replicate, OpenAI, Vertex AI)

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)
![MyPy](https://img.shields.io/badge/MyPy-Strict-blue)
![Tests](https://img.shields.io/badge/Tests-Passing-success)

---

## 📋 О Проекте

**AI Provider SDK** — type-safe библиотека для унифицированной работы с различными AI-провайдерами. Обеспечивает единый интерфейс для генерации изображений, текста и других AI-задач независимо от конкретного провайдера.

### Ключевые Особенности

✅ **Multi-Provider**: Replicate, OpenAI, Vertex AI  
✅ **Type-Safe**: MyPy strict mode, Pydantic schemas  
✅ **Async/Sync**: Поддержка обоих режимов  
✅ **Provider Abstraction**: Легко добавить нового провайдера  
✅ **Credits Calculation**: Автоматический расчёт стоимости  
✅ **Rate Limiting**: Встроенная защита от превышения лимитов  

---

## 🚀 Установка

```bash
pip install ai-provider-sdk
```

### Требования

- Python 3.11+
- API ключи провайдеров (Replicate, OpenAI, Vertex AI)

---

## 📖 Быстрый Старт

### 1. Базовое Использование

```python
from ai_provider_sdk import ProviderFactory, ProviderConfig

# Конфигурация
config = ProviderConfig(
    replicate_api_key="YOUR_REPLICATE_KEY",
    openai_api_key="YOUR_OPENAI_KEY"
)

# Создание фабрики провайдеров
factory = ProviderFactory(config)

# Получение провайдера
provider = factory.get_provider("replicate")

# Генерация изображения
result = await provider.generate_image(
    model="flux-1.1-pro",
    prompt="A beautiful sunset over mountains",
    width=1024,
    height=768
)

print(f"Image URL: {result.image_url}")
print(f"Generation time: {result.generation_time}s")
print(f"Credits spent: {result.credits_spent}")
```

### 2. Работа с Разными Провайдерами

```python
# Replicate
replicate = factory.get_provider("replicate")
image = await replicate.generate_image(
    model="flux-1.1-pro",
    prompt="Cyberpunk city"
)

# OpenAI
openai = factory.get_provider("openai")
text = await openai.generate_text(
    model="gpt-4",
    prompt="Write a poem about AI"
)

# Vertex AI (Google Cloud)
vertex = factory.get_provider("vertex_ai")
image = await vertex.generate_image(
    model="imagen-3.0-generate-001",
    prompt="Abstract art"
)
```

### 3. Синхронный Режим

```python
from ai_provider_sdk import SyncProviderFactory

factory = SyncProviderFactory(config)
provider = factory.get_provider("replicate")

# Синхронная генерация
result = provider.generate_image(
    model="flux-1.1-pro",
    prompt="Mountain landscape"
)
```

---

## 🛠️ API Документация

### ProviderFactory

#### `get_provider(name: str)`
Получить провайдера по имени

**Параметры**:
- `name` (str): Имя провайдера ("replicate", "openai", "vertex_ai")

**Возвращает**: Provider объект

```python
provider = factory.get_provider("replicate")
```

### BaseProvider

Все провайдеры наследуются от `BaseProvider` и реализуют единый интерфейс:

#### `generate_image()`
Генерация изображения

**Параметры**:
- `model` (str): ID модели
- `prompt` (str): Текстовый промпт
- `width` (int, optional): Ширина изображения
- `height` (int, optional): Высота изображения
- `**kwargs`: Дополнительные параметры модели

**Возвращает**: `GenerationResult` объект

```python
result = await provider.generate_image(
    model="flux-1.1-pro",
    prompt="A cat in space",
    width=1024,
    height=1024,
    num_inference_steps=50
)
```

#### `generate_text()`
Генерация текста

**Параметры**:
- `model` (str): ID модели
- `prompt` (str): Текстовый промпт
- `max_tokens` (int, optional): Максимум токенов
- `**kwargs`: Дополнительные параметры

**Возвращает**: `TextGenerationResult` объект

```python
result = await provider.generate_text(
    model="gpt-4",
    prompt="Explain quantum computing",
    max_tokens=500
)
```

---

## 📦 Pydantic Schemas

### GenerationResult
```python
class GenerationResult(BaseModel):
    image_url: str
    generation_time: float
    credits_spent: float
    model: str
    provider: str
```

### TextGenerationResult
```python
class TextGenerationResult(BaseModel):
    text: str
    tokens_used: int
    generation_time: float
    credits_spent: float
    model: str
```

### ModelConfig
```python
class ModelConfig(BaseModel):
    id: str
    provider: str
    category: str  # "txt2img", "txt2txt", "img2img"
    credits_per_generation: float
    parameters: Dict[str, Any]
```

---

## 🔧 Продвинутое Использование

### Кастомные Модели

```python
from ai_provider_sdk import ModelConfig

# Добавить кастомную модель
custom_model = ModelConfig(
    id="my-custom-model",
    provider="replicate",
    category="txt2img",
    credits_per_generation=0.05,
    parameters={
        "prompt": {"type": "string"},
        "width": {"type": "integer", "default": 1024},
        "height": {"type": "integer", "default": 1024}
    }
)

factory.register_model(custom_model)
```

### Rate Limiting

```python
from ai_provider_sdk import RateLimitConfig

rate_limit = RateLimitConfig(
    requests_per_minute=60,
    requests_per_hour=1000
)

provider = factory.get_provider(
    "replicate",
    rate_limit=rate_limit
)
```

### Credits Tracking

```python
# Получить стоимость генерации
cost = provider.calculate_cost(
    model="flux-1.1-pro",
    width=1024,
    height=1024
)

print(f"Estimated cost: {cost} credits")

# Генерация с проверкой баланса
if user_balance >= cost:
    result = await provider.generate_image(...)
    user_balance -= result.credits_spent
```

### Error Handling

```python
from ai_provider_sdk import (
    ProviderError,
    InsufficientCreditsError,
    RateLimitError,
    ModelNotFoundError
)

try:
    result = await provider.generate_image(...)
except InsufficientCreditsError:
    print("Недостаточно кредитов")
except RateLimitError as e:
    print(f"Превышен лимит. Retry after {e.retry_after}s")
except ModelNotFoundError:
    print("Модель не найдена")
except ProviderError as e:
    print(f"Ошибка провайдера: {e.message}")
```

---

## 🧪 Тестирование

```bash
# Установить dev зависимости
pip install -e ".[dev]"

# Запустить тесты
pytest

# С покрытием
pytest --cov=ai_provider_sdk

# Type checking
mypy ai_provider_sdk --strict
```

---

## 📚 Примеры

### Пример 1: Batch Generation

```python
from ai_provider_sdk import BatchGenerator

prompts = [
    "A sunset over mountains",
    "A cyberpunk city",
    "Abstract art"
]

batch = BatchGenerator(provider)
results = await batch.generate_images(
    model="flux-1.1-pro",
    prompts=prompts,
    max_concurrent=3  # Параллельно 3 генерации
)

for result in results:
    print(f"Generated: {result.image_url}")
```

### Пример 2: Provider Fallback

```python
from ai_provider_sdk import ProviderChain

# Попробовать провайдеров по порядку
chain = ProviderChain([
    factory.get_provider("replicate"),
    factory.get_provider("openai"),
    factory.get_provider("vertex_ai")
])

# Автоматический fallback при ошибке
result = await chain.generate_image(
    model="flux-1.1-pro",
    prompt="Mountain landscape"
)
```

### Пример 3: Кеширование Результатов

```python
from ai_provider_sdk import CachedProvider

# Провайдер с кешированием
cached_provider = CachedProvider(
    provider=factory.get_provider("replicate"),
    cache_ttl=3600  # 1 час
)

# Первый запрос — генерация
result1 = await cached_provider.generate_image(
    model="flux-1.1-pro",
    prompt="A cat"
)

# Второй запрос с тем же промптом — из кеша
result2 = await cached_provider.generate_image(
    model="flux-1.1-pro",
    prompt="A cat"
)

print(f"From cache: {result2.from_cache}")  # True
```

---

## 🔐 Безопасность

### Управление Секретами

```python
import os
from ai_provider_sdk import ProviderConfig

# ✅ Хорошо: секреты из переменных окружения
config = ProviderConfig(
    replicate_api_key=os.getenv("REPLICATE_API_KEY"),
    openai_api_key=os.getenv("OPENAI_API_KEY"),
    vertex_ai_project=os.getenv("GCP_PROJECT_ID")
)

# ❌ Плохо: хардкод секретов
config = ProviderConfig(
    replicate_api_key="r8_abc123...",  # НИКОГДА так не делайте!
)
```

---

## 🎯 Поддерживаемые Провайдеры

| Провайдер | Текст | Изображения | Видео | Аудио |
|-----------|-------|-------------|-------|-------|
| **Replicate** | ✅ | ✅ | ✅ | ✅ |
| **OpenAI** | ✅ | ✅ | ❌ | ✅ |
| **Vertex AI** | ✅ | ✅ | ❌ | ❌ |

---

## 📖 Связанные Проекты

- [AI Tools Platform Case Study](https://github.com/kets-kets/portfolio-case-studies/blob/main/ai-tools-platform.md) — Production использование этой библиотеки
- [mini-ai-playground](https://github.com/kets-kets/mini-ai-playground) — Демо-проект с интеграцией

---

## 🤝 Вклад в Проект

Приветствуются Pull Requests! Особенно интересны:

- Новые провайдеры (Anthropic, Midjourney, etc.)
- Улучшение документации
- Примеры использования
- Bug fixes

---

## 📄 Лицензия

MIT License - см. [LICENSE](LICENSE)

---

## 👤 Автор

**kets**  
- GitHub: [@kets-kets](https://github.com/kets-kets)  
- Telegram: [@ketsdpt](https://t.me/ketsdpt)

---

## 📦 О Библиотеке

Библиотека выделена из production проекта AI Tools Platform и адаптирована для публичного использования.
