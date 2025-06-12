
import asyncio
import json
import os
from aiogram import Bot, Dispatcher, F
from aiogram.enums import ParseMode
from aiogram.types import Message, InlineKeyboardButton, InlineKeyboardMarkup
from aiogram.filters import Command, CommandStart
from aiogram.client.default import DefaultBotProperties
from datetime import datetime

# === CONFIGURATION ===
API_TOKEN = "7626524597:AAEJf6jUA-tvBkA6sUchOuV3LE0ViMpmT7w"
ALLOWED_GROUP_ID = -1002892180435
VIP_USER_ID = 7394822106
DATA_FILE = "user_data.json"

# === BOT SETUP ===
bot = Bot(API_TOKEN, default=DefaultBotProperties(parse_mode=ParseMode.HTML))
dp = Dispatcher()

# === UTILS ===
def load_data():
    if not os.path.exists(DATA_FILE):
        with open(DATA_FILE, "w") as f:
            json.dump({}, f)
    with open(DATA_FILE, "r") as f:
        return json.load(f)

def save_data(data):
    with open(DATA_FILE, "w") as f:
        json.dump(data, f, indent=2)


REQUIRED_CHANNEL = "@oxunovff"

async def is_user_subscribed(user_id: int) -> bool:
    try:
        member = await bot.get_chat_member(REQUIRED_CHANNEL, user_id)
        return member.status in ["member", "administrator", "creator"]
    except Exception:
        return False

def join_keyboard():

    return InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text="📢 Join Channel", url="https://t.me/oxunovff")],
    ])

def vip_keyboard():
    return InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text="📢 Join Channel", url="https://t.me/oxunovff")],
        [InlineKeyboardButton(text="💎 Buy VIP", url="https://t.me/oxunjonov28")],
    ])

# === COMMANDS ===
@dp.message(CommandStart(deep_link=True))
async def start_with_ref(update: Message, command: CommandStart):
    data = load_data()
    user_id = str(update.from_user.id)
    ref_id = command.args

    if user_id not in data:
        data[user_id] = {
            "likes": {"CIS": 0, "RUS": 0},
            "referrals": [],
            "vip": False
        }

        if ref_id and ref_id != user_id and ref_id in data:
            if user_id not in data[ref_id]["referrals"]:
                data[ref_id]["likes"]["CIS"] += 1  # reward referrer
                data[ref_id]["referrals"].append(user_id)

    save_data(data)
    await update.answer("🤖 Botga xush kelibsiz! Sizning hisobingiz tayyor.")

@dp.message(Command("start"))

@dp.message(Command("start"))
async def start(update: Message):
    if not await is_user_subscribed(update.from_user.id):
        await update.answer("❗ Iltimos, <b>@oxunovff</b> kanaliga a'zo bo'ling!", reply_markup=join_keyboard())
        return

    await update.answer("🤖 Salom! Like botga hush kelibsiz!", reply_markup=join_keyboard())

@dp.message(Command("like"))

@dp.message(Command("like"))
async def like(update: Message):
    if not await is_user_subscribed(update.from_user.id):
        await update.answer("❗ Iltimos, <b>@oxunovff</b> kanaliga a'zo bo'ling!", reply_markup=join_keyboard())
        return

    user_id = str(update.from_user.id)
    data = load_data()

    if user_id not in data:
        await update.answer("Iltimos, avval /start buyrug'ini yuboring.")
        return

    user_data = data[user_id]
    cis_likes = user_data["likes"]["CIS"]
    rus_likes = user_data["likes"]["RUS"]
    vip_status = "✅" if user_data["vip"] else "❌"

    text = (
        f"❤️ Sizda {cis_likes} ta CIS like, {rus_likes} ta RUS like bor.
"
        f"💎 VIP holatingiz: {vip_status}
"
        f"👥 Referallaringiz soni: {len(user_data['referrals'])}"
    )
    await update.answer(text)

# === START POLLING ===
async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
