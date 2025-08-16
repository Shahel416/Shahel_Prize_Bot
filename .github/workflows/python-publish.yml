#!/data/data/com.termux/files/usr/bin/python3
import os
import json
import random
from datetime import datetime
from typing import List

from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import (
    ApplicationBuilder, CommandHandler, ContextTypes,
    MessageHandler, filters, CallbackQueryHandler,
    ConversationHandler
)

# ================= CONFIG =================
DATA_FILE = "participants.json"
WINNERS_FILE = "winners.json"

BOT_TOKEN = "8340679576:AAGKFBSxqNO9CG2sAD-ZXgRvWTOqyO83hbA"
ADMIN_IDS = {6010980146, 6610220146}

PASSWORD1 = "12345"
PASSWORD2 = "12345"

PWD1, PWD2 = range(2)
DRAW_DATE = "N/A"

# ================= DATA UTILS =================
def load_participants() -> List[int]:
    if not os.path.exists(DATA_FILE):
        return []
    with open(DATA_FILE, "r") as f:
        try:
            return json.load(f)
        except:
            return []

def save_participants(participants: List[int]):
    with open(DATA_FILE, "w") as f:
        json.dump(participants, f)

def load_winners() -> List[dict]:
    if not os.path.exists(WINNERS_FILE):
        return []
    with open(WINNERS_FILE, "r") as f:
        try:
            return json.load(f)
        except:
            return []

def save_winners(winners: List[dict]):
    with open(WINNERS_FILE, "w") as f:
        json.dump(winners, f)

def is_admin(user_id: int) -> bool:
    return user_id in ADMIN_IDS

# ================= START ====================
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    keyboard, all_joined = await build_condition_buttons(user, context)
    await update.message.reply_markdown(
        "🔐 *Attention...!*\n\n"
        " ⚠️ You must fulfill all conditions to join the raffle.\n\n"
        "📣 After fulfill all conditions give command /start than click Continue.\n\n"
        "📢 Make sure you are a member of all our official Telegram channels.\n\n"
        "👇 Check conditions below and proceed...!",
        reply_markup=InlineKeyboardMarkup(keyboard)
    )

# ================= BUTTON HELPERS ==================
async def build_condition_buttons(user, context):
    required_groups = [
        "@shahel_gaming",
        "@shahel_gaming_movies",
        "@shahel_gaming_animes",
        "@shahel_gaming"
    ]
    buttons = []
    all_joined = True
    for grp in required_groups:
        try:
            member = await context.bot.get_chat_member(grp, user.id)
            status_emoji = "✅" if member.status not in ["left", "kicked"] else "❌"
            if status_emoji == "❌":
                all_joined = False
            buttons.append([InlineKeyboardButton(f"{status_emoji} {grp}", url=f"https://t.me/{grp[1:]}")])
        except:
            all_joined = False
            buttons.append([InlineKeyboardButton(f"❌ {grp}", url=f"https://t.me/{grp[1:]}")])

    continue_text = "✅ Continue" if all_joined else "❌ Continue"
    buttons.append([InlineKeyboardButton(continue_text, callback_data="continue_raffle")])
    return buttons, all_joined

# ================= BUTTON HANDLER ====================
async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user = query.from_user
    data = query.data

    if data == "continue_raffle":
        all_joined = True
        required_groups = [
            "@shahel_gaming",
            "@shahel_gaming_movies",
            "@shahel_gaming_animes",
            "@shahel_gaming"
        ]
        for grp in required_groups:
            try:
                member = await context.bot.get_chat_member(grp, user.id)
                if member.status in ["left", "kicked"]:
                    all_joined = False
                    break
            except:
                all_joined = False
                break

        if not all_joined:
            await query.edit_message_text("⚠️ You must join all required groups to proceed!")
            return ConversationHandler.END
        else:
            await show_main_menu(query)
            return ConversationHandler.END

    elif data == "join":
        await query.edit_message_text(
            "🔐 Enter Password 1:",
            reply_markup=InlineKeyboardMarkup([[InlineKeyboardButton("🔙 Back", callback_data="continue_raffle")]])
        )
        return PWD1

    elif data == "leave":
        participants = load_participants()
        if user.id in participants:
            participants.remove(user.id)
            save_participants(participants)
            await query.edit_message_text(
                f"❌ {user.first_name} left the raffle.",
                reply_markup=InlineKeyboardMarkup([[InlineKeyboardButton("🔙 Back", callback_data="continue_raffle")]])
            )
        else:
            await query.edit_message_text(
                "❌ You are not registered.",
                reply_markup=InlineKeyboardMarkup([[InlineKeyboardButton("🔙 Back", callback_data="continue_raffle")]])
            )
        return ConversationHandler.END

    elif data == "participants":
        participants = load_participants()
        if not participants:
            text = "📋 *No participants yet.*"
        else:
            text = "📋 *Participants List:*\n" + "\n".join(
                [f"{i+1}. `User ID: {uid}`" for i, uid in enumerate(participants)]
            )
        await query.edit_message_text(
            text, parse_mode="Markdown",
            reply_markup=InlineKeyboardMarkup([[InlineKeyboardButton("🔙 Back", callback_data="continue_raffle")]])
        )
        return ConversationHandler.END

    elif data == "status":
        winners = load_winners()
        if not winners:
            text = "📊 *No winners yet.*"
        else:
            text = "📊 *Winners History:*\n\n"
            for idx, win in enumerate(winners, start=1):
                time_str = win.get("time", "Unknown time")
                name = win.get("name", "Unknown")
                uid = win.get("user_id", "Unknown ID")
                text += f"🎖️ {idx}. {name} (`{uid}`)\n    🕒 Won on: _{time_str}_\n\n"
        await query.edit_message_text(
            text, parse_mode="Markdown",
            reply_markup=InlineKeyboardMarkup([[InlineKeyboardButton("🔙 Back", callback_data="continue_raffle")]])
        )
        return ConversationHandler.END

    elif data == "info":
        buttons = [
            [InlineKeyboardButton("🔹 Telegram Channels", callback_data="show_telegram")],
            [InlineKeyboardButton("▶️ YouTube", url="https://cutt.ly/JeBisrmI")],
            [InlineKeyboardButton("📸 Instagram", url="https://cutt.ly/geBiacDy")],
            [InlineKeyboardButton("🎵 TikTok", url="https://cutt.ly/oeBiaAHI")],
            [InlineKeyboardButton("🎮 Twitch", url="https://cutt.ly/keBia1qa")],
            [InlineKeyboardButton("📘 Facebook", url="https://cutt.ly/ceBisay9")],
            [InlineKeyboardButton("🔙 Back", callback_data="continue_raffle")]
        ]
        reply_markup = InlineKeyboardMarkup(buttons)
        await query.edit_message_text("✨ *Shahel Gaming Raffle Info* ✨", parse_mode="Markdown", reply_markup=reply_markup)
        return ConversationHandler.END

    elif data == "show_telegram":
        buttons = [
            [InlineKeyboardButton("@shahel_gaming", url="https://t.me/shahel_gaming")],
            [InlineKeyboardButton("@shahel_gaming_movies", url="https://t.me/shahel_gaming_movies")],
            [InlineKeyboardButton("@shahel_gaming_animes", url="https://t.me/shahel_gaming_animes")],
            [InlineKeyboardButton("@shahel_gaming", url="https://t.me/shahel_gaming")],
            [InlineKeyboardButton("🔙 Back", callback_data="info")]
        ]
        reply_markup = InlineKeyboardMarkup(buttons)
        await query.edit_message_text("📢 *Our Telegram Channels:*", parse_mode="Markdown", reply_markup=reply_markup)
        return ConversationHandler.END

# ================= PASSWORD HANDLERS ==================
async def password1(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text.strip()
    if text != PASSWORD1:
        await update.message.reply_text("🚫 Incorrect Password 1! Try again.")
        return PWD1
    await update.message.reply_text("✅ Password 1 verified. Enter Password 2.")
    return PWD2

async def password2(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    text = update.message.text.strip()
    if text != PASSWORD2:
        await update.message.reply_text("🚫 Incorrect Password 2! Try again.")
        return PWD2

    participants = load_participants()
    if user.id not in participants:
        participants.append(user.id)
        save_participants(participants)

    buttons = [[InlineKeyboardButton("🔙 Back to Raffle", callback_data="continue_raffle")]]
    await update.message.reply_text(
        f"🎉 Congratulations, {user.first_name}! You joined the raffle.",
        reply_markup=InlineKeyboardMarkup(buttons)
    )
    return ConversationHandler.END

async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("❎ Password entry cancelled.")
    return ConversationHandler.END

# ================= MAIN MENU =================
async def show_main_menu(query):
    raffle_info_text = (
        "🎮 *Welcome to Shahel Gaming Raffle* 🎮\n\n"
        "✨ *Join our exciting raffle and win amazing prizes!* ✨\n\n"
        "🔐 *Passwords (2) required to join.*\n\n"
        f"📅 Draw Date: {DRAW_DATE}\n\n"
        "📢 Get the passwords from our official channel: [@shahel_gaming](https://t.me/shahel_gaming)\n\n"
    )
    keyboard = [
        [InlineKeyboardButton("🎉 JOIN", callback_data="join")],
        [InlineKeyboardButton("❌ LEAVE", callback_data="leave")],
        [InlineKeyboardButton("📋 Participants", callback_data="participants")],
        [InlineKeyboardButton("📊 Status", callback_data="status")],
        [InlineKeyboardButton("ℹ️ Info", callback_data="info")],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await query.edit_message_text(
        raffle_info_text + "\n👇 Choose an action below:",
        parse_mode="Markdown",
        reply_markup=reply_markup
    )

# ================= ADMIN COMMANDS ==================
async def draw_winners(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    if not is_admin(user.id):
        await update.message.reply_text("❌ You are not authorized to use this command.")
        return

    args = context.args
    if not args or not args[0].isdigit():
        await update.message.reply_text("Usage: /draw <number_of_winners>")
        return

    num_winners = int(args[0])
    participants = load_participants()

    if not participants:
        await update.message.reply_text("⚠️ No participants available for draw.")
        return

    if num_winners > len(participants):
        await update.message.reply_text(f"⚠️ Only {len(participants)} participants available.")
        return

    selected = random.sample(participants, num_winners)
    winners_list = []

    text = "🎉 *Fresh Draw Winners* 🎉\n\n"

    for idx, uid in enumerate(selected, start=1):
        try:
            member = await context.bot.get_chat(uid)
            name = member.first_name
        except:
            name = f"User {uid}"
        winners_list.append({
            "user_id": uid,
            "name": name,
            "time": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        })
        text += f"{idx}. {name} (`{uid}`)\n"

    save_winners(winners_list)
    save_participants([])  # Fresh draw removes participants
    await update.message.reply_markdown(text)

async def reset_all(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    if not is_admin(user.id):
        await update.message.reply_text("❌ You are not authorized to use this command.")
        return
    save_participants([])
    save_winners([])
    await update.message.reply_text("✅ All data has been reset. Raffle is fresh now.")

async def set_draw_date(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    global DRAW_DATE
    if not is_admin(user.id):
        await update.message.reply_text("❌ You are not authorized to use this command.")
        return
    args = context.args
    if not args:
        await update.message.reply_text("Usage: /setdate YYYY-MM-DD")
        return
    DRAW_DATE = args[0]
    await update.message.reply_text(f"✅ Draw date updated to {DRAW_DATE}")

async def change_password1(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    global PASSWORD1
    if not is_admin(user.id):
        await update.message.reply_text("❌ You are not authorized to use this command.")
        return
    args = context.args
    if not args:
        await update.message.reply_text("Usage: /changepwd1 NEWPWD")
        return
    PASSWORD1 = args[0]
    await update.message.reply_text("✅ Password 1 updated successfully.")

async def change_password2(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    global PASSWORD2
    if not is_admin(user.id):
        await update.message.reply_text("❌ You are not authorized to use this command.")
        return
    args = context.args
    if not args:
        await update.message.reply_text("Usage: /changepwd2 NEWPWD")
        return
    PASSWORD2 = args[0]
    await update.message.reply_text("✅ Password 2 updated successfully.")

# ================= MAIN =================
def main():
    app = ApplicationBuilder().token(BOT_TOKEN).build()

    # ConversationHandler for password entry
    conv_handler = ConversationHandler(
        entry_points=[CallbackQueryHandler(button_handler, pattern="^join$")],
        states={
            PWD1: [MessageHandler(filters.TEXT & ~filters.COMMAND, password1)],
            PWD2: [MessageHandler(filters.TEXT & ~filters.COMMAND, password2)],
        },
        fallbacks=[CommandHandler("cancel", cancel)],
        per_user=True,
        per_chat=False,
    )

    # Start command
    app.add_handler(CommandHandler("start", start))

    # Admin commands
    app.add_handler(CommandHandler("draw", draw_winners))
    app.add_handler(CommandHandler("reset", reset_all))
    app.add_handler(CommandHandler("setdate", set_draw_date))
    app.add_handler(CommandHandler("changepwd1", change_password1))
    app.add_handler(CommandHandler("changepwd2", change_password2))

    # Password conversation handler
    app.add_handler(conv_handler)

    # CallbackQuery handler for all buttons
    app.add_handler(CallbackQueryHandler(button_handler))

    print("Bot is running...")
    app.run_polling()

if __name__ == "__main__":
    main()
