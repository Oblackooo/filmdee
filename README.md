import requests
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters
import os

# Твой API-ключ от TMDb (берём из переменных окружения)
TMDb_API_KEY = os.environ.get('TMDB_API_KEY')
BOT_TOKEN = os.environ.get('BOT_TOKEN')

async def start(update: Update, context):
    await update.message.reply_text('Привет! Напиши название фильма, и я найду его подробное описание на русском.')

async def search_movie(update: Update, context):
    movie_title = update.message.text
    url = f'https://api.themoviedb.org/3/search/movie?api_key={TMDb_API_KEY}&query={movie_title}&language=ru-RU'

    try:
        # Поиск фильма
        search_response = requests.get(url)
        search_data = search_response.json()

        if search_data['results']:
            movie_id = search_data['results'][0]['id']  # Берем первый результат
            # Получаем детали фильма
            details_url = f'https://api.themoviedb.org/3/movie/{movie_id}?api_key={TMDb_API_KEY}&language=ru-RU'
            details_response = requests.get(details_url)
            details_data = details_response.json()

            title = details_data['title']
            release_year = details_data['release_date'][:4] if details_data['release_date'] else 'Год неизвестен'
            overview = details_data['overview'] or 'Описания нет'

            message = f"{title} ({release_year})\n{overview}"
            await update.message.reply_text(message)
        else:
            await update.message.reply_text('Фильм не найден! Попробуй ввести точное название на русском или английском, например, "Матрица" или "The Matrix".')
    except Exception as e:
        await update.message.reply_text('Произошла ошибка. Попробуй позже.')

def main():
    application = Application.builder().token(BOT_TOKEN).build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, search_movie))

    application.run_polling()

if __name__ == '__main__':
    main()
