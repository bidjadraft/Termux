




```python
import os
import pathlib
import requests
import json
import re
import asyncio
from telegram import Update, constants
from telegram.error import TimedOut
from telegram.ext import (
    ApplicationBuilder, CommandHandler, MessageHandler,
    ContextTypes, ConversationHandler, filters
)

# اسم ملف تخزين التوكن
TOKEN_FILE = "tokens.txt"

def save_tokens(bot_token, gemini_api_key):
    with open(TOKEN_FILE, "w", encoding="utf-8") as f:
        f.write(f"BOT_TOKEN={bot_token}\n")
        f.write(f"GEMINI_API_KEY={gemini_api_key}\n")

def load_tokens():
    if not os.path.exists(TOKEN_FILE):
        return None, None

    bot_token, gemini_api_key = None, None
    with open(TOKEN_FILE, "r", encoding="utf-8") as f:
        for line in f:
            if line.startswith("BOT_TOKEN="):
                bot_token = line.strip().split("=", 1)[1]
            elif line.startswith("GEMINI_API_KEY="):
                gemini_api_key = line.strip().split("=", 1)[1]
    return bot_token, gemini_api_key


def check_bot_token(token):
    url = f"https://api.telegram.org/bot{token}/getMe"
    try:
        response = requests.get(url, timeout=5)
        return response.status_code == 200 and response.json().get("ok", False)
    except Exception:
        return False


def check_gemini_key(key):
    test_url = f"https://generativelanguage.googleapis.com/v1beta/models?key={key}"
    try:
        resp = requests.get(test_url, timeout=5)
        return resp.status_code == 200
    except:
        return False


def prompt_for_token():
    while True:
        bot_token = input("BOT TOKEN: ").strip()
        if check_bot_token(bot_token):
            print("Bot token is valid.")
            break
        else:
            print("Invalid bot token. Please try again.")
    return bot_token


def prompt_for_gemini_key():
    while True:
        key = input("GEMINI API: ").strip()
        if check_gemini_key(key):
            print("Gemini API key is valid.")
            break
        else:
            print("Invalid Gemini API key. Please try again.")
    return key


# تحميل التوكنات والاحتفاظ بها في المتغيرات
TELEGRAM_BOT_TOKEN, GEMINI_API_KEY = load_tokens()

if not TELEGRAM_BOT_TOKEN or not check_bot_token(TELEGRAM_BOT_TOKEN):
    TELEGRAM_BOT_TOKEN = prompt_for_token()

if not GEMINI_API_KEY or not check_gemini_key(GEMINI_API_KEY):
    GEMINI_API_KEY = prompt_for_gemini_key()

save_tokens(TELEGRAM_BOT_TOKEN, GEMINI_API_KEY)


# عناوين URL وإعدادات الموديل
UPLOAD_START_URL = f"https://generativelanguage.googleapis.com/upload/v1beta/files?key={GEMINI_API_KEY}"
GEMINI_MODEL = "gemini-2.5-flash"
GEMINI_URL = f"https://generativelanguage.googleapis.com/v1beta/models/{GEMINI_MODEL}:generateContent?key={GEMINI_API_KEY}"

WAITING_ACCEPT_FILE = 1
WAITING_QUERY = 2
MAX_TELEGRAM_TEXT_LENGTH = 4096
STATE_FILE = "session_state.json"

def load_state():
    if os.path.exists(STATE_FILE):
        with open(STATE_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return {}

def save_state(state):
    with open(STATE_FILE, "w", encoding="utf-8") as f:
        json.dump(state, f)

all_state = load_state()

def start_upload_session(file_path, mime_type):
    file_size = os.path.getsize(file_path)
    metadata = {"file": {"displayName": os.path.basename(file_path)}}
    headers = {
        "X-Goog-Upload-Protocol": "resumable",
        "X-Goog-Upload-Command": "start",
        "X-Goog-Upload-Header-Content-Length": str(file_size),
        "X-Goog-Upload-Header-Content-Type": mime_type,
        "Content-Type": "application/json",
    }
    response = requests.post(UPLOAD_START_URL, headers=headers, json=metadata)
    if response.status_code != 200:
        print("فشل بدء جلسة الرفع:", response.text)
        return None
    return response.headers.get("X-Goog-Upload-URL")

def upload_file_data(upload_url, file_path, mime_type):
    with open(file_path, "rb") as f:
        data = f.read()
    headers = {
        "X-Goog-Upload-Command": "upload, finalize",
        "X-Goog-Upload-Offset": "0",
        "Content-Type": mime_type,
        "Content-Length": str(len(data)),
    }
    response = requests.post(upload_url, headers=headers, data=data)
    if response.status_code != 200:
        print("فشل رفع الملف:", response.text)
        return None
    file_info = response.json()
    return file_info.get("file", {}).get("uri")

def send_request_to_gemini(file_uri, prompt_text):
    payload = {
        "contents": [
            {
                "parts": [
                    {"text": prompt_text},
                    {"fileData": {"fileUri": file_uri, "mimeType": "application/pdf"}},
                ]
            }
        ]
    }
    headers = {"Content-Type": "application/json"}

    try:
        response = requests.post(GEMINI_URL, headers=headers, json=payload)
        if response.status_code == 200:
            data = response.json()
            try:
                return data["candidates"][0]["content"]["parts"][0]["text"]
            except (IndexError, KeyError):
                return "تعذر فهم رد Gemini."
        elif response.status_code == 503:
            return "خطأ: الخدمة غير متوفرة حالياً. يرجى المحاولة لاحقاً."
        else:
            return f"خطأ من Gemini API: {response.status_code}"
    except requests.exceptions.RequestException as e:
        return f"فشل الطلب: {e}. يرجى المحاولة لاحقاً."

def underline_page_numbers(text):
    return re.sub(r'(الصفحة\s+\d+)', r'<u>\1</u>', text)

def remove_duplicates_from_text(text):
    sentences = re.split(r'(?<!\w\.\w.)(?<![A-Z][a-z]\.)(?<=\.|\?|\!)\s', text)
    unique_sentences = []
    seen = set()
    for sentence in sentences:
        if sentence not in seen:
            unique_sentences.append(sentence)
            seen.add(sentence)
    return ' '.join(unique_sentences)

def sanitize_html_chunk(chunk):
    open_tags = re.findall(r'<([ubi])>', chunk)
    closed_tags = re.findall(r'</([ubi])>', chunk)

    tags_to_close = []
    for tag in open_tags:
        if open_tags.count(tag) > closed_tags.count(tag):
            tags_to_close.append(tag)
    for tag in tags_to_close:
        chunk += f'</{tag}>'
    return chunk

def reformat_page_numbers(text):
    return re.sub(r'الصفحة:\s*(\d+):', r'الصفحة (\1):', text)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    chat_id = str(update.effective_chat.id)
    saved = all_state.get(chat_id)
    if saved and saved.get("file_uri"):
        context.user_data["file_uri"] = saved["file_uri"]
        await update.message.reply_text("يرجى كتابة موضوع البحث داخل الملف الآن.")
        return WAITING_QUERY
    else:
        await update.message.reply_text("مرحبًا! أرسل لي ملف PDF للبدء.")
        return WAITING_ACCEPT_FILE

async def handle_document(update: Update, context: ContextTypes.DEFAULT_TYPE):
    chat_id = str(update.effective_chat.id)
    file_obj = update.message.document
    filename = file_obj.file_name if file_obj.file_name else f"{file_obj.file_id}.pdf"
    MAX_SIZE = 20 * 1024 * 1024  # 20 ميجابايت

    if file_obj.file_size > MAX_SIZE:
        await update.message.reply_text("الملف كبير جدًا. يرجى حفظه محليًا ثم إعادة إرسال الملف.")
        return WAITING_ACCEPT_FILE

    temp_dir = context.bot_data.get("temp_dir", "temp")
    os.makedirs(temp_dir, exist_ok=True)
    temp_file_path = pathlib.Path(temp_dir) / filename
    file = await context.bot.get_file(file_obj.file_id)
    await file.download_to_drive(str(temp_file_path))

    # عرض شريط تحميل متحرك
    message = await update.message.reply_text("[░░░░░░░░░░] 0%")
    bar_length = 10  # طول شريط التحميل
    percent = 0

    # تحديث شريط التحميل وهمي تدريجيًا
    while percent < 90:
        percent += 13  # للاقتراب من النسبة المطلوبة
        if percent > 90:
            percent = 90

        num_filled = int((percent / 100) * bar_length)
        num_empty = bar_length - num_filled
        bar = "▓" * num_filled + "░" * num_empty
        text = f"[{bar}] {percent}%"
        try:
            await message.edit_text(text)
        except Exception:
            pass

        await asyncio.sleep(0.5)

    upload_url = start_upload_session(temp_file_path, "application/pdf")
    if not upload_url:
        await message.edit_text("فشل بدء رفع الملف إلى Google AI.")
        return WAITING_ACCEPT_FILE
    file_uri = upload_file_data(upload_url, temp_file_path, "application/pdf")
    if not file_uri:
        await message.edit_text("فشل رفع بيانات الملف إلى Google AI.")
        return WAITING_ACCEPT_FILE

    # إكمال شريط التحميل للـ100% ثم حذف الرسالة
    percent = 100
    num_filled = bar_length
    bar = "▓" * num_filled
    final_text = f"[{bar}] {percent}%"
    try:
        await message.edit_text(final_text)
        await asyncio.sleep(0.5)
        await message.delete()
    except Exception:
        pass

    context.user_data["file_uri"] = file_uri
    all_state[chat_id] = {"file_uri": file_uri}
    save_state(all_state)

    await update.message.reply_text("تم رفع الملف بنجاح، يمكنك الآن كتابة موضوع البحث.")
    return WAITING_QUERY

async def receive_query(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not update.message or not update.message.text:
        return WAITING_QUERY

    chat_id = str(update.effective_chat.id)
    file_uri = context.user_data.get("file_uri") or all_state.get(chat_id, {}).get("file_uri")
    if not file_uri:
        await update.message.reply_text("لم يتم رفع ملف بعد، أرسل ملف PDF أولاً.")
        return WAITING_ACCEPT_FILE

    query_text = update.message.text.strip()
    if not query_text:
        await update.message.reply_text("من فضلك أرسل نصًا صالحًا للبحث.")
        return WAITING_QUERY

    try:
        message = await update.message.reply_text("[░░░░░░░░░░] 0%")
    except TimedOut:
        print("Timeout occurred while sending progress message")
        return WAITING_QUERY

    previous_text = ""

    async def fetch_gemini_result():
        prompt_text = (
            f"Extract the information related to the query from the document. "
            f"The response must be extremely concise and direct. Do not add any introductory or concluding sentences. "
            f"For each piece of information, use the exact format: '• الصفحة: المحتوى', followed by a blank line (a new line and a new line again). "
            f"Use Western numerals (like 1, 2, 3) for the page numbers, not Arabic ones (like ١, ٢, ٣). "
            f"Ensure the text is grammatically correct and free of spelling errors, correcting any that you find. The query is:\n{query_text}"
        )
        loop = asyncio.get_event_loop()
        return await loop.run_in_executor(None, send_request_to_gemini, file_uri, prompt_text)

    gemini_task = asyncio.create_task(fetch_gemini_result())

    percent = 0

    while not gemini_task.done() and percent < 95:
        percent += 1
        filled_blocks = "▓" * (percent // 10)
        empty_blocks = "░" * (10 - (percent // 10))
        new_text = f"[{filled_blocks}{empty_blocks}] {percent}%"
        if new_text != previous_text:
            try:
                await message.edit_text(new_text)
                previous_text = new_text
            except TimedOut:
                print("Timeout occurred while editing progress message")
        await asyncio.sleep(0.2)

    while not gemini_task.done():
        filled_blocks = "▓" * 9 + "░"
        new_text = f"[{filled_blocks}] 95%"
        if new_text != previous_text:
            try:
                await message.edit_text(new_text)
                previous_text = new_text
            except TimedOut:
                print("Timeout occurred while editing progress message")
        await asyncio.sleep(3)

    percent = 100
    filled_blocks = "▓" * 10
    new_text = f"[{filled_blocks}] {percent}%"
    try:
        await message.edit_text(new_text)
    except TimedOut:
        print("Timeout occurred while editing progress message")

    response_text = gemini_task.result()

    try:
        await message.delete()
    except Exception:
        pass

    response_text = reformat_page_numbers(response_text)
    cleaned_text = remove_duplicates_from_text(response_text)
    parts = [part.strip() for part in cleaned_text.split('•') if part.strip()]
    formatted_text = '• ' + '\n\n• '.join(parts)
    formatted_text = underline_page_numbers(formatted_text)

    if len(formatted_text) > MAX_TELEGRAM_TEXT_LENGTH:
        chunks = [formatted_text[i:i + MAX_TELEGRAM_TEXT_LENGTH] for i in range(0, len(formatted_text), MAX_TELEGRAM_TEXT_LENGTH)]
        for chunk in chunks:
            sanitized_chunk = sanitize_html_chunk(chunk)
            await update.message.reply_text(sanitized_chunk, parse_mode=constants.ParseMode.HTML)
    else:
        await update.message.reply_text(formatted_text, parse_mode=constants.ParseMode.HTML)

    return WAITING_QUERY

async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("تم إلغاء العملية.")
    return ConversationHandler.END

if __name__ == "__main__":
    conv_handler = ConversationHandler(
        entry_points=[CommandHandler("start", start)],
        states={
            WAITING_ACCEPT_FILE: [MessageHandler(filters.Document.PDF, handle_document)],
            WAITING_QUERY: [
                MessageHandler(filters.TEXT & ~filters.COMMAND, receive_query),
                MessageHandler(filters.Document.PDF, handle_document),
            ],
        },
        fallbacks=[CommandHandler("cancel", cancel)],
    )

    app = ApplicationBuilder().token(TELEGRAM_BOT_TOKEN).build()
    app.add_handler(conv_handler)

    print("Bot is running...")
    app.run_polling()


```