import time
import instaloader
import requests
import json

# إعداد بيانات تسجيل الدخول
username = "aithajyassine3@gmail.com"
password = "Yaithaj12@1"

# إعداد بيانات Meta API
ACCESS_TOKEN = 'YOUR_ACCESS_TOKEN'  # استبدل هذا بالتوكن الخاص بك
INSTAGRAM_ACCOUNT_ID = '974030418112384'  # استبدل بمعرف حسابك على إنستغرام

# إنشاء كائن Instaloader مع تعريف User-Agent مخصص
bot = instaloader.Instaloader(
    user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36"
)

try:
    # محاولة تحميل جلسة سابقة
    bot.load_session_from_file(username)
    print("تم تحميل الجلسة المخزنة بنجاح!")
except FileNotFoundError:
    print("لا توجد جلسة مخزنة. محاولة تسجيل الدخول...")
    try:
        # تسجيل الدخول باستخدام كلمة المرور
        bot.context.log("محاولة تسجيل الدخول...")
        bot.login(user=username, passwd=password)
        print("تم تسجيل الدخول بنجاح!")
        # حفظ الجلسة
        bot.save_session_to_file()
    except instaloader.exceptions.BadCredentialsException:
        print("خطأ: اسم المستخدم أو كلمة المرور غير صحيحة.")
    except instaloader.exceptions.InvalidArgumentException:
        print("خطأ: تحقق من اسم المستخدم أو الإعدادات.")
    except instaloader.exceptions.ConnectionException as e:
        print(f"خطأ في الاتصال: {e}")
    except Exception as e:
        print("فشل تسجيل الدخول: ", str(e))

# الرد الذكي باستخدام اللغة واللهجات المختلفة
from langdetect import detect

def smart_reply(message):
    try:
        language = detect(message)
        if language == 'ar':
            return "أنا أحمد، روبوت ذكي يمكنني التحدث بجميع اللهجات العربية والإنجليزية. أقدم معلومات دقيقة عن عيادات أبولونيا وأسهل الطرق للحجز مع أطبائنا المتخصصين."
        elif language == 'en':
            return "I am Ahmed, a smart assistant who can speak in all Arabic dialects and English. I provide accurate information about Apollonia Clinics and assist with booking appointments with our specialists."
        else:
            return "عذرًا، لا أستطيع التعرف على اللغة. هل يمكنك الكتابة بالعربية أو الإنجليزية؟"
    except Exception:
        return "عذرًا، لا أستطيع التعرف على اللغة. هل يمكنك الكتابة بالعربية أو الإنجليزية؟"

# META API START
# وظيفة لإرسال الرسائل عبر Meta API
def send_meta_reply(message, recipient_id):
    url = f"https://graph.facebook.com/v16.0/{INSTAGRAM_ACCOUNT_ID}/messages"
    headers = {
        "Authorization": f"Bearer {ACCESS_TOKEN}",
        "Content-Type": "application/json"
    }
    data = {
        "recipient": {"id": recipient_id},
        "message": {"text": message}
    }
    response = requests.post(url, headers=headers, json=data)
    return response.json()
# META API END

# مثال على الرد التلقائي
print("أهلاً وسهلاً بكم في عيادات أبولونيا! أنا أحمد، مساعدكم الذكي. كيف يمكنني مساعدتكم اليوم؟")

while True:
    try:
        # استقبال الرسائل (محاكاة الرسائل)
        incoming_message = input("رسالة واردة: ")
        if incoming_message.lower() == 'exit':
            print("شكراً لتواصلكم معنا في عيادات أبولونيا. نتمنى لكم يوماً سعيداً! لا تترددوا في التواصل معنا مستقبلاً.")
            break

        # التحقق إذا كان المستخدم يريد حجز موعد
        if 'حجز' in incoming_message or 'موعد' in incoming_message:
            reply = "لإتمام عملية الحجز، يرجى تزويدنا باسمك ورقم هاتفك حتى يتواصل معك أحد موظفي خدمة العملاء." \
                if detect(incoming_message) == 'ar' else "To complete the booking process, please provide your name and phone number so that one of our customer service representatives can contact you."
        else:
            # تحليل الرسالة وإعداد الرد الذكي حول أبولونيا فقط
            reply = smart_reply(incoming_message)

        print("الرد: ", reply)

        # META API SEND EXAMPLE
        # إرسال الرسالة عبر Meta API (تجريبي)
        recipient_id = "USER_ID"  # استبدل بمعرف المستخدم
        meta_response = send_meta_reply(reply, recipient_id)
        print("استجابة META API: ", meta_response)

    except Exception as e:
        print("حدث خطأ: ", str(e))
