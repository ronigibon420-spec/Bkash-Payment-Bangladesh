import asyncio
from aiogram import Bot, Dispatcher, types
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.utils import executor

# 🔐 তোমার বট টোকেন
BOT_TOKEN = "8248993557:AAHl_SOA17uiTxW7E50M-32Fe0TZi8mPfCE"

# 👑 অ্যাডমিন গ্রুপ আইডি
ADMIN_GROUP_ID = -4814959376

# 📢 Force Join চ্যানেল লিস্ট
CHANNELS = [
    "https://t.me/BNTFree",
    "https://t.me/TkIncomebd570",
    "https://t.me/bnrfree",
    "https://t.me/BkashPaymentBangladesh"
]

# ইউজার ডেটা (ডেমো হিসেবে লোকালি)
users = {}

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher(bot)

# 🔹 Start কমান্ড
@dp.message_handler(commands=['start'])
async def start_cmd(message: types.Message):
    user_id = message.from_user.id

    if user_id not in users:
        users[user_id] = {"balance": 0, "verified": False, "referrals": 0}

    text = (
        "🎉 Nagad Bangladesh Bot এ আপনাকে স্বাগতম!\n\n"
        "💸 Join + Verify করলে পাবেন 720 টাকা বোনাস।\n"
        "👥 প্রতি রেফারে পাবেন 75 টাকা (Join+Verify এর পরে)।\n"
        "🏧 1500 টাকা হলে Nagad দিয়ে withdraw করতে পারবেন।"
    )

    btn = InlineKeyboardMarkup(row_width=1)
    btn.add(InlineKeyboardButton("📢 Join Channel", url=CHANNELS[0]))
    btn.add(InlineKeyboardButton("✅ Verify", callback_data="verify_channels"))

    await message.answer(text, reply_markup=btn)

# 🔹 Verify বাটন হ্যান্ডলার
@dp.callback_query_handler(lambda c: c.data == "verify_channels")
async def verify_channels(callback: types.CallbackQuery):
    user_id = callback.from_user.id
    users[user_id]["verified"] = True
    users[user_id]["balance"] += 720

    btn = InlineKeyboardMarkup(row_width=3)
    btn.add(
        InlineKeyboardButton("💰 Balance", callback_data="balance"),
        InlineKeyboardButton("👥 Refer", callback_data="refer"),
        InlineKeyboardButton("🏧 Withdraw", callback_data="withdraw")
    )

    await callback.message.edit_text(
        f"✅ আপনি সফলভাবে ভেরিফাই করেছেন!\n৭২০ টাকা আপনার ব্যালেন্সে যোগ হয়েছে।\n\n"
        f"👉 নিচ থেকে একটি অপশন বেছে নিন:",
        reply_markup=btn
    )

# 🔹 Balance
@dp.callback_query_handler(lambda c: c.data == "balance")
async def balance_info(callback: types.CallbackQuery):
    user_id = callback.from_user.id
    bal = users[user_id]["balance"]
    await callback.answer()
    await callback.message.answer(f"💰 আপনার বর্তমান ব্যালেন্স: {bal} টাকা")

# 🔹 Refer
@dp.callback_query_handler(lambda c: c.data == "refer")
async def refer_link(callback: types.CallbackQuery):
    user_id = callback.from_user.id
    ref_link = f"https://t.me/{(await bot.me).username}?start={user_id}"
    await callback.answer()
    await callback.message.answer(f"👥 আপনার রেফার লিংক:\n{ref_link}")

# 🔹 Withdraw
@dp.callback_query_handler(lambda c: c.data == "withdraw")
async def withdraw_request(callback: types.CallbackQuery):
    user_id = callback.from_user.id
    bal = users[user_id]["balance"]
    if bal < 1500:
        await callback.message.answer("❌ আপনার ব্যালেন্স 1500 টাকার নিচে। Withdraw সম্ভব নয়।")
    else:
        await callback.message.answer("✅ আপনার Withdraw অনুরোধ অ্যাডমিনকে পাঠানো হয়েছে।")
        await bot.send_message(
            ADMIN_GROUP_ID,
            f"📤 নতুন Withdraw অনুরোধ:\n👤 User ID: {user_id}\n💰 ব্যালেন্স: {bal} টাকা"
        )

# 🔹 রেফারাল বোনাস (যখন /start <ref_id> দিয়ে যোগ হয়)
@dp.message_handler(lambda message: message.text.startswith("/start ") and len(message.text.split()) > 1)
async def start_with_ref(message: types.Message):
    ref_id = int(message.text.split()[1])
    user_id = message.from_user.id
    if user_id == ref_id:
        await message.answer("❌ নিজেকে রেফার করা সম্ভব নয়!")
        return

    if user_id not in users:
        users[user_id] = {"balance": 0, "verified": True, "referrals": 0}
        users[ref_id]["balance"] += 75
        users[ref_id]["referrals"] += 1
        await message.answer("✅ আপনি রেফার লিংক দিয়ে যোগ হয়েছেন!")
        await bot.send_message(ref_id, f"👥 আপনি ১টি রেফার পেয়েছেন! ৭৫ টাকা যোগ হলো।")

# 🔹 Run the Bot
if __name__ == "__main__":
    print("🤖 Bot is running...")
    executor.start_polling(dp, skip_updates=True)# Bkash-Payment-Bangladesh
