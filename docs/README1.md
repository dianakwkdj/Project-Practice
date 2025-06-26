Для проекта **«Басманные хроники. Путешествие через вселенные»** я разработала чат-бота в Telegram, который будет помогать при выполнении заданий:
### **Чат-бот «Помощник по игре»**  
**Цель:**  
Поддержка игроков, прохождение квестов, подсказки.  

**Функционал:**  
- **Гайды и подсказки** – как пройти сложные моменты.  
- **Персонажи игры в боте** – NPC могут "общаться" с игроком вне игры.  
- **Таймеры событий** – напоминания о новых локациях или обновлениях.  
- **Конкурсы и викторины** – дополнительные баллы за знание истории.  

**Пример:**  
> Игрок застрял на квесте → бот предлагает подсказку или мини-викторину для разблокировки следующего шага.  

#### **Технологический стек**
| Компонент       | Технологии                     |
|-----------------|-------------------------------|
| Frontend (игра) | Godot 4, GDScript, C#        |
| Backend         | Python (FastAPI), PostgreSQL  |
| Бот            | Aiogram 3, Redis (для FSM)   |
| Интеграции      | Музейный API (GraphQL)        |


### **📌 Функционал бота:**   
1. **Подсказки по игре** – помощь в прохождении квестов.  
2. **Диалоги с NPC** – общение с персонажами вне игры.  
3. **Инвентарь** – просмотр коллекционных предметов.  (в доработке)
4. **Исторические справки** – доп. информация о локациях.  


#### **Таблица функционала бота**
| Модуль          | Методы                    | Описание                          |
|-----------------|--------------------------|-----------------------------------|
| `/quests`       | GET /active, POST /start | Управление квестами               |
| `/inventory`    | GET /items, POST /use    | Работа с инвентарём               |
| `/history`      | GET /facts?location=id   | Исторические справки              |
| `/npc`          | POST /talk               | Диалоговая система                |

---

### **🔧 Технологии:**  
- **Язык:** Python (библиотеки `aiogram` или `python-telegram-bot`)  
- **База данных:** SQLite / PostgreSQL (для хранения прогресса)   

---
После создания бота в BotFather, он выдал мне токен, который я использовала для написания кода на языке Python.
### **📝 Код, который использовался для создания бота на Python: **  

```mermaid
flowchart TB
    subgraph Telegram
        A[Бот] -->|Webhook| B[Сервер]
    end
    
    subgraph Cloud
        B --> C[Godot Server]
        C --> D[(DB)]
        C --> E[Museum API]
    end
    
    A --> F[Игроки]
    C --> F
```


#### **1. Настройка бота**  
```python
from aiogram import Bot, Dispatcher, types, F
from aiogram.filters import Command
import sqlite3

bot = Bot(token="8100940938:AAG-hEM6aAwh7pb0C48ko9V2wnP7ZwSTyeY")
dp = Dispatcher()

# Подключение к БД
conn = sqlite3.connect('basmannye.db')
cursor = conn.cursor()

# Создание таблицы для пользователей
cursor.execute('''CREATE TABLE IF NOT EXISTS users
               (user_id INTEGER PRIMARY KEY, username TEXT, inventory TEXT)''')
conn.commit()
```

#### **2. Команды и кнопки**  
```python
# Главное меню
@dp.message(Command("start"))
async def start(message: types.Message):
    kb = [
        [types.KeyboardButton(text="🎮 Подсказки по игре")],
        [types.KeyboardButton(text="📜 Исторические справки")],
        [types.KeyboardButton(text="💬 Персонажи")],
        [types.KeyboardButton(text="📦 Мой инвентарь")]
    ]
    keyboard = types.ReplyKeyboardMarkup(keyboard=kb, resize_keyboard=True)
    await message.answer("Добро пожаловать в «Басманные хроники»! Чем помочь?", reply_markup=keyboard)
```

#### **3. Раздел «Подсказки»**  
```python
@dp.message(F.text == "🎮 Подсказки по игре")
async def game_hints(message: types.Message):
    await message.answer(
        "Выберите локацию:",
        reply_markup=types.InlineKeyboardMarkup(inline_keyboard=[
            [types.InlineKeyboardButton(text="Станция «Бауманская»", callback_data="hint_baumanskaya")],
            [types.InlineKeyboardButton(text="Лаборатория", callback_data="hint_lab")]
        ])
    )

@dp.callback_query(F.data.startswith("hint_"))
async def send_hint(callback: types.CallbackQuery):
    location = callback.data.split("_")[1]
    hints = {
        "baumanskaya": "🔍 Осмотрите левую платформу – там спрятан ключ от старого тоннеля.",
        "lab": "🧪 Поговорите с учёным – он даст задание на восстановление механизма."
    }
    await callback.message.answer(hints.get(location, "Подсказка не найдена."))
```

#### **4. Раздел «Персонажи»**  
```python
@dp.message(F.text == "💬 Персонажи")
async def npc_chat(message: types.Message):
    await message.answer(
        "С кем хотите поговорить?",
        reply_markup=types.InlineKeyboardMarkup(inline_keyboard=[
            [types.InlineKeyboardButton(text="Профессор", callback_data="npc_professor")],
            [types.InlineKeyboardButton(text="Механик", callback_data="npc_mechanic")]
        ])
    )

@dp.callback_query(F.data.startswith("npc_"))
async def npc_dialogue(callback: types.CallbackQuery):
    npc = callback.data.split("_")[1]
    dialogues = {
        "professor": "👨‍🔬 «Ваши открытия изменят ход истории! Проверьте архивные записи.»",
        "mechanic": "🔧 «Этот механизм сломан… Найдите шестерёнку в старом депо.»"
    }
    await callback.message.answer(dialogues.get(npc, "Персонаж не отвечает."))
```

#### **5. Запуск бота**  
```python
async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

---
#### **6. Пример работы бота**  
![Screenshot1](https://github.com/dianakwkdj/Project-Practice/blob/master/docs/img/1.png)

![Screenshot1](https://github.com/dianakwkdj/Project-Practice/blob/master/docs/img/2.png)

Ссылка на бота: https://web.telegram.org/a/#8100940938
