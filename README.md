# introproject.bot
#my
import telebot
from telebot import types

bot = telebot.TeleBot('6837895288:AAF5f28qJwtBKkTGvzCIWyDDZY_jnW2cXBo')
owner_chat_id = '820042687'

@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.reply_to(message, "Добро пожаловать! Для записи на курс подготовки к ОРТ, пожалуйста, укажите ваше ФИО:")
    bot.register_next_step_handler(message, process_name_step)

def process_name_step(message):
    try:
        chat_id = message.chat.id
        name = message.text

        user_dict[chat_id] = {'name': name}

        msg = bot.send_message(chat_id, "Сколько вам лет?")
        bot.register_next_step_handler(msg, process_age_step)

    except Exception as e:
        bot.reply_to(message, 'Ошибка! Пожалуйста, укажите ваше ФИО.')

def process_age_step(message):
    try:
        chat_id = message.chat.id
        age = message.text

        user_dict[chat_id]['age'] = age

        msg = bot.send_message(chat_id, "В каком классе вы учитесь?")
        bot.register_next_step_handler(msg, process_grade_step)

    except Exception as e:
        bot.reply_to(message, 'Ошибка! Пожалуйста, укажите ваш возраст.')

def process_grade_step(message):
    try:
        chat_id = message.chat.id
        grade = message.text

        user_dict[chat_id]['grade'] = grade

        msg = bot.send_message(chat_id, "Введите ваш контактный номер телефона (10 цифр):")
        bot.register_next_step_handler(msg, process_phone_step)

    except Exception as e:
        bot.reply_to(message, 'Ошибка! Пожалуйста, укажите ваш класс.')

def process_phone_step(message):
    try:
        chat_id = message.chat.id
        phone = message.text


        if phone.isdigit() and len(phone) == 10:
            user_dict[chat_id]['phone'] = phone

            markup = types.ReplyKeyboardMarkup(one_time_keyboard=True)
            markup.add(types.KeyboardButton('Русский'), types.KeyboardButton('Кыргызский'))

            msg = bot.send_message(chat_id, "Выберите язык подготовки:", reply_markup=markup)
            bot.register_next_step_handler(msg, process_language_step)
        else:
            raise ValueError('Некорректный формат номера телефона.')

    except ValueError as ve:
        bot.reply_to(message, str(ve))
    except Exception as e:
        bot.reply_to(message, 'Ошибка! Пожалуйста, введите ваш номер телефона (10 цифр).')

def process_language_step(message):
    try:
        chat_id = message.chat.id
        language = message.text

        user_dict[chat_id]['language'] = language


        applicant_info = f"Заявка на курс подготовки к ОРТ:\n\n" \
                         f"ФИО: {user_dict[chat_id]['name']}\n" \
                         f"Возраст: {user_dict[chat_id]['age']}\n" \
                         f"Класс: {user_dict[chat_id]['grade']}\n" \
                         f"Язык подготовки: {user_dict[chat_id]['language']}\n" \
                         f"Телефон: {user_dict[chat_id]['phone']}"

        bot.send_message(owner_chat_id, applicant_info)

        bot.send_message(chat_id, "Спасибо за вашу заявку! Мы свяжемся с вами в ближайшее время.")

    except Exception as e:
        bot.reply_to(message, 'Ошибка! Не удалось обработать вашу заявку.')

if __name__ == '__main__':
    user_dict = {}
    while True:
        try:
            bot.polling(none_stop=True)
        except Exception as e:

            print(f"Ошибка: {str(e)}")
            bot.stop_polling()
