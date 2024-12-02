from telegram import Update, ChatPermissions
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext
from dotenv import load_dotenv
import os

# Load token dari file .env
load_dotenv()
TOKEN = os.getenv("BOT_TOKEN")

if not TOKEN:
    raise ValueError("Bot token tidak ditemukan di file .env")

# Fungsi untuk memulai bot
def start(update: Update, context: CallbackContext):
    update.message.reply_text(
        "Halo! Saya adalah bot moderasi grup. Gunakan /help untuk melihat daftar perintah."
    )

# Fungsi untuk menampilkan daftar perintah
def help_command(update: Update, context: CallbackContext):
    help_text = """
    Daftar Perintah Bot Moderasi:
    /mute [balas pesan] - Membungkam pengguna
    /unmute [balas pesan] - Menghapus pembungkaman pengguna
    /kick [balas pesan] - Menendang pengguna
    /antispam - Menghapus pesan spam
    /antilink on/off - Mengatur filter link
    """
    update.message.reply_text(help_text)

# Fungsi untuk membungkam pengguna
def mute(update: Update, context: CallbackContext):
    if update.message.reply_to_message:
        user_id = update.message.reply_to_message.from_user.id
        chat_id = update.message.chat_id
        permissions = ChatPermissions(
            can_send_messages=False,
            can_send_media_messages=False,
            can_send_polls=False,
            can_send_other_messages=False,
            can_add_web_page_previews=False,
        )
        context.bot.restrict_chat_member(chat_id, user_id, permissions)
        update.message.reply_text("Pengguna telah dibungkam.")

# Fungsi untuk menghapus pembungkaman pengguna
def unmute(update: Update, context: CallbackContext):
    if update.message.reply_to_message:
        user_id = update.message.reply_to_message.from_user.id
        chat_id = update.message.chat_id
        permissions = ChatPermissions(
            can_send_messages=True,
            can_send_media_messages=True,
            can_send_polls=True,
            can_send_other_messages=True,
            can_add_web_page_previews=True,
        )
        context.bot.restrict_chat_member(chat_id, user_id, permissions)
        update.message.reply_text("Pembungkaman pengguna telah dihapus.")

# Fungsi untuk menendang pengguna
def kick(update: Update, context: CallbackContext):
    if update.message.reply_to_message:
        user_id = update.message.reply_to_message.from_user.id
        chat_id = update.message.chat_id
        context.bot.kick_chat_member(chat_id, user_id)
        update.message.reply_text("Pengguna telah ditendang dari grup.")

# Fungsi untuk menghapus pesan spam
def antispam(update: Update, context: CallbackContext):
    if "spam" in update.message.text.lower():
        update.message.delete()

# Fungsi untuk mengatur filter link
antilink_status = {"enabled": False}

def antilink(update: Update, context: CallbackContext):
    global antilink_status
    args = context.args
    if args and args[0].lower() == "on":
        antilink_status["enabled"] = True
        update.message.reply_text("Fitur Anti-Link diaktifkan.")
    elif args and args[0].lower() == "off":
        antilink_status["enabled"] = False
        update.message.reply_text("Fitur Anti-Link dinonaktifkan.")
    else:
        update.message.reply_text("Gunakan /antilink on atau /antilink off.")

def filter_links(update: Update, context: CallbackContext):
    if antilink_status["enabled"]:
        if "http://" in update.message.text or "https://" in update.message.text:
            update.message.delete()
            update.message.reply_text("Link tidak diperbolehkan di grup ini.")

# Fungsi utama untuk menjalankan bot
def main():
    updater = Updater(TOKEN)
    dispatcher = updater.dispatcher

    # Daftar perintah dan handler
    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("help", help_command))
    dispatcher.add_handler(CommandHandler("mute", mute))
    dispatcher.add_handler(CommandHandler("unmute", unmute))
    dispatcher.add_handler(CommandHandler("kick", kick))
    dispatcher.add_handler(CommandHandler("antispam", antispam))
    dispatcher.add_handler(CommandHandler("antilink", antilink))
    dispatcher.add_handler(MessageHandler(Filters.text & ~Filters.command, filter_links))
    dispatcher.add_handler(MessageHandler(Filters.text, antispam))

    # Jalankan bot
    updater.start_polling()
    updater.idle()

if __name__ == "__main__":
    main()
