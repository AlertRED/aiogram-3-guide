---
title: Знакомство с aiogram
description: Знакомство с aiogram
---

# Знакомство с aiogram
## Терминология {: id="glossary" }

Прежде, чем рассматривать aiogram, введём некоторые термины, чтобы в дальнейшем не путаться:

* ЛС — личные сообщения, в контексте бота это диалог один-на-один с пользователем, а не группа/канал.
* Чат — общее название для ЛС, групп, супергрупп и каналов.
* Апдейт — любое событие из [этого списка](https://core.telegram.org/bots/api#update): 
сообщение, редактирование сообщения, колбэк, инлайн-запрос, платёж, добавление бота в группу и т.д. 
* Хэндлер — асинхронная функция, которая получает непосредственно из Telegram очередной апдейт 
и обрабатывает его.
* Диспетчер — объект, занимающийся выбором хэндлера для обработки очередного апдейта.
* Роутер — аналогично диспетчеру, но отвечает за подмножество множества хэндлеров. 
**Можно сказать, что диспетчер — это корневой роутер**.
* Фильтр — выражение, которое обычно возвращает True или False и влияет на то, будет вызван хэндлер или нет.
* Мидлварь — прослойка, которая вклинивается в обработку апдейтов. 

## Установка {: id="installation" }

Для начала давайте создадим каталог для бота, организуем там virtual environment (далее venv) и
установим библиотеку [aiogram](https://github.com/aiogram/aiogram).  
Проверим, что установлен Python версии 3.9 (если вы знаете, что установлен 3.9 и выше, можете пропустить этот раздел):

```plain
[groosha@main lesson_01]$ python3.9
Python 3.9.9 (main, Jan 11 2022, 16:35:07) 
[GCC 11.1.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
[groosha@main lesson_01]$ 
```

Теперь создадим файл `requirements.txt`, в котором укажем используемую нами версию aiogram.
!!! important "О версиях aiogram"
    В этой главе используется aiogram **3.x**, перед началом работы рекомендую заглянуть в 
    [канал релизов](https://t.me/aiogram_live) библиотеки и проверить наличие более новой версии. Подойдёт любая 
    более новая, начинающаяся с цифры 3, поскольку aiogram 2.x более рассматриваться не будет и считается устаревшим.

```plain
[groosha@main 01_quickstart]$ python3.9 -m venv venv
[groosha@main 01_quickstart]$ echo "aiogram==3.0.0b1" > requirements.txt
[groosha@main 01_quickstart]$ source venv/bin/activate
(venv) [groosha@main 01_quickstart]$ pip install --pre -r requirements.txt 
# ...здесь куча строк про установку...
Successfully installed ...тут длинный список...
[groosha@main 01_quickstart]$
```

Обратите внимание на префикс "venv" в терминале. Он указывает, что мы находимся в виртуальном окружении с именем "venv".
Проверим, что внутри venv вызов команды `python` указывает на всё тот же Python 3.9:  
```plain
(venv) [groosha@main 01_quickstart]$ python
Python 3.9.9 (main, Jan 11 2022, 16:35:07) 
[GCC 11.1.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
(venv) [groosha@main 01_quickstart]$ deactivate 
[groosha@main 01_quickstart]$ 
```

Последней командой `deactivate` мы вышли из venv, чтобы он нам не мешал. 

## Первый бот {: id="hello-world" }

Давайте создадим файл `bot.py` с базовым шаблоном бота на aiogram:
```python title="bot.py"
import asyncio
import logging
from aiogram import Bot, Dispatcher, types

# Включаем логирование, чтобы не пропустить важные сообщения
logging.basicConfig(level=logging.INFO)
# Объект бота
bot = Bot(token="12345678:AaBbCcDdEeFfGgHh")
# Диспетчер
dp = Dispatcher()

# Хэндлер на команду /start
@dp.message(commands="start")
async def cmd_start(message: types.Message):
    await message.answer("Hello!")

# Запуск процесса поллинга новых апдейтов
async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
```

Первое, на что нужно обратить внимание: aiogram — асинхронная библиотека, поэтому ваши хэндлеры тоже должны быть асинхронными, 
а перед вызовами методов API нужно ставить ключевое слово **await**, т.к. эти вызовы возвращают [корутины](https://docs.python.org/3/library/asyncio-task.html#coroutines).

!!! info "Асинхронное программирование в Python"
    Не стоит пренебрегать официальной документацией!  
    Прекрасный туториал по asyncio доступен [на сайте Python](https://docs.python.org/3/library/asyncio-task.html).

Если вы в прошлом работали с какой-то другой библиотекой для Telegram, например, pyTelegramBotAPI, то концепция
хэндлеров (обработчиков событий) вам сразу станет понятна, разница лишь в том, что в aiogram хэндлерами управляет диспетчер.  
Диспетчер регистрирует функции-обработчики, дополнительно ограничивая перечень вызывающих их событий через фильтры. 
После получения очередного апдейта (события от Telegram), диспетчер выберет нужную функцию обработки, подходящую по всем 
фильтрам, например, «обработка сообщений, являющихся изображениями, в чате с ID икс и с длиной подписи игрек». Если две 
функции имеют одинаковые по логике фильтры, то будет вызвана та, что зарегистрирована раньше.

Чтобы зарегистрировать функцию как обработчик сообщений, нужно сделать одно из двух действий:  
1. Навесить на неё [декоратор](https://devpractice.ru/python-lesson-19-decorators/), как в примере выше. 
С различными типами декораторов мы познакомимся позднее.
2. Напрямую вызвать метод регистрации у диспетчера или роутера.

Рассмотрим следующий код: 
```python
# Хэндлер на команду /test1
@dp.message(commands="test1")
async def cmd_test1(message: types.Message):
    await message.reply("Test 1")

# Хэндлер на команду /test2
async def cmd_test2(message: types.Message):
    await message.reply("Test 2")
```

Давайте запустим с ним бота:  
![Команда /test2 не работает](images/quickstart/l01_1.jpg)

Хэндлер `cmd_test2` не сработает, т.к. диспетчер о нём не знает. Исправим эту ошибку 
и отдельно зарегистрируем функцию:
```python
# Хэндлер на команду /test2
async def cmd_test2(message: types.Message):
    await message.reply("Test 2")

# Где-то в другом месте, например, в функции main():
dp.message.register(cmd_test2, commands="test2")
```

Снова запустим бота:  
![Обе команды работают](images/quickstart/l01_2.jpg)

## Синтаксический сахар {: id="sugar" }

Для того, чтобы сделать код чище и читабельнее, aiogram расширяет возможности стандартных объектов Telegram.
Например, вместо `bot.send_message(...)` можно написать `message.answer(...)` или `message.reply(...)`. В последних
двух случаях не нужно подставлять `chat_id`, подразумевается, что он такой же, как и в исходном сообщении.  
Разница между `answer` и `reply` простая: первый метод просто отправляет сообщение в тот же чат, второй делает "ответ" на 
сообщение из `message`:
```python
@dp.message(commands="answer")
async def cmd_answer(message: types.Message):
    await message.answer("Это простой ответ")


@dp.message(commands="reply")
async def cmd_reply(message: types.Message):
    await message.reply('Это ответ с "ответом"')
```
![Разница между message.answer() и message.reply()](images/quickstart/l01_3.jpg)

Более того, для большинства типов сообщений есть вспомогательные методы вида 
"answer_{type}" или "reply_{type}", например:
```python
@dp.message(commands="dice")
async def cmd_dice(message: types.Message):
    await message.answer_dice(emoji="🎲")
```

!!! info "что значит 'message: types.Message' ?"
    Python является интерпретируемым языком с [сильной, но динамической типизацией](https://habr.com/ru/post/161205/),
    поэтому встроенная проверка типов, как, например, в C++ или Java, отсутствует. Однако начиная с версии 3.5 
    в языке появилась поддержка [подсказок типов](https://docs.python.org/3/library/typing.html), благодаря которой
    различные чекеры и IDE вроде PyCharm анализируют типы используемых значений и подсказывают
    программисту, если он передаёт что-то не то. В данном случае подсказка `types.Message` соообщает
    PyCharm-у, что переменная `message` имеет тип `Message`, описанный в модуле `types` библиотеки
    aiogram (см. импорты в начале кода). Благодаря этому IDE может на лету подсказывать атрибуты и функции.

При вызове команды `/dice` бот отправит в тот же чат игральный кубик. Разумеется, если его надо отправить в какой-то
другой чат, то придётся по-старинке вызывать `await bot.send_dice(...)`. Но объект `bot` (экземпляр класса Bot) может быть 
недоступен в области видимости конкретной функции. В aiogram 3.x объект бота, которому пришёл апдейт, неявно 
прокидывается в хэндлер и его можно достать как аргумент `bot`. Предположим, вы хотите по команде `/dice` 
отправлять кубик не в тот же чат, а в канал с ID -100123456789. Перепишем предыдущую функцию:

```python
@dp.message(commands="dice")
async def cmd_dice(message: types.Message, bot: Bot):
    await bot.send_dice(-100123456789, emoji="🎲")
```

На этом мы закончим знакомство с библиотекой, а в следующих главах рассмотрим другие "фишки" aiogram и Telegram Bot API.
