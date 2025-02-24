import telebot
import requests

TOKEN = "7703468310:AAHd_IyHyHnvmvQOWRSkUcyCfYHMFvC2XLk"
bot = telebot.TeleBot(TOKEN)

@bot.message_handler(commands=['start'])
def start(message):
    bot.reply_to(message, "أهلاً بك! أرسل اسم مستخدم تيك توك لتحليل الحساب.")

@bot.message_handler(func=lambda message: True)
def handle_username(message):
    username = message.text.strip()

    # إرسال خيارات للمستخدم كأزرار
    markup = telebot.types.ReplyKeyboardMarkup(one_time_keyboard=True, resize_keyboard=True)
    markup.add("TikTok Beta", "TikTok")
    
    bot.send_message(message.chat.id, "اختر نوع التحليل:", reply_markup=markup)
    bot.register_next_step_handler(message, choose_option, username)

def choose_option(message, username):
    option = message.text.strip().lower()
    
    if option == "tiktok beta":
        search_tiktok_beta(message, username)
    elif option == "tiktok":
        search_tiktok(message, username)
    else:
        bot.send_message(message.chat.id, "❌ الخيار غير صحيح، حاول مرة أخرى.")
        handle_username(message)

def fetch_tiktok_data(username):
    url = f"https://www.tikwm.com/api/user/info?unique_id={username}"
    
    try:
        response = requests.get(url).json()
        if response.get("code") == 0 and "data" in response:
            user = response["data"]["user"]
            stats = response["data"]["stats"]
            return {
                "nickname": user.get("nickname", "غير متوفر"),
                "email": user.get("email", "غير متوفر"),
                "followers": stats.get("followerCount", 0),
                "likes": stats.get("heartCount", 0),
                "videos": stats.get("videoCount", 0),
                "avatar": user.get("avatarLarger", ""),
                "account_link": f"https://www.tiktok.com/@{username}"
            }
        else:
            return None
    except Exception as e:
        return str(e)

def send_tiktok_report(message, username, beta=False):
    data = fetch_tiktok_data(username)
    
    if isinstance(data, str):
        bot.send_message(message.chat.id, f"❌ حدث خطأ أثناء جلب البيانات: {data}")
        return

    if data is None:
        bot.send_message(message.chat.id, "❌ لم يتم العثور على الحساب، تحقق من الاسم.")
        return

    report = (
        f"📊 تحليل حساب @{username}:\n"
        f"👤 الاسم: {data['nickname']}\n"
        f"📧 البريد الإلكتروني: {data['email']}\n"
        f"👥 المتابعون: {data['followers']}\n"
        f"❤️ الإعجابات: {data['likes']}\n"
        f"🎥 عدد الفيديوهات: {data['videos']}\n"
        f"🔗 رابط الحساب: {data['account_link']}\n"
    )

    bot.send_photo(message.chat.id, data['avatar'], caption=report)

def search_tiktok(message, username):
    send_tiktok_report(message, username, beta=False)

def search_tiktok_beta(message, username):
    send_tiktok_report(message, username, beta=True)

bot.polling()
