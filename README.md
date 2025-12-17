# mnt-price-bot
import os
import asyncio
import aiohttp
from datetime import datetime
from telegram import Bot

TELEGRAM_TOKEN = os.environ.get('8272842347:AAE1VuNpesYhHLIqpz1E0ROl_XVRZtxOT88')
CHAT_ID = os.environ.get('950105749')

async def main():
    print("🤖 Бот запускается...")
    
    if not TELEGRAM_TOKEN or not CHAT_ID:
        print("❌ Ошибка: не установлены TELEGRAM_TOKEN или CHAT_ID")
        return
    
    bot = Bot(token=8272842347:AAE1VuNpesYhHLIqpz1E0ROl_XVRZtxOT88)
    
    while True:
        try:
            # Получаем цену BTC с Binance
            async with aiohttp.ClientSession() as session:
                API_URL = "https://api.bybit.com/v2/public/tickers?symbol=MNTUSD"
                async with session.get(url) as response:
                    data = await response.json()
                    price = float(data['price'])
            
            # Формируем сообщение
            time_now = datetime.now().strftime("%H:%M:%S")
            message = f"""
💰 <b>MNT PRICE</b>
┌────────────────
│ <b>${price:,.2f}</b>
│ 🕐 {time_now}
│ 📊 Binance
└────────────────
            """
            
            # Отправляем в Telegram
            await bot.send_message(
                chat_id=CHAT_ID,
                text=message,
                parse_mode='HTML'
            )
            
            print(f"✅ Отправлено: ${price} в {time_now}")
            
        except Exception as e:
            print(f"❌ Ошибка: {e}")
        
        # Ждём 60 секунд
        await asyncio.sleep(60)

if __name__ == "__main__":
    asyncio.run(main())
