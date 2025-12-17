import os
import asyncio
import aiohttp
from datetime import datetime
from telegram import Bot

TELEGRAM_TOKEN = os.environ.get('8272842347:AAE1VuNpesYhHLIqpz1E0ROl_XVRZtxOT88')
CHAT_ID = os.environ.get('950105749')

async def get_mnt_price():
    """Получаем цену MNT из разных источников"""
    apis = [
        ("Gate.io", "https://api.gateio.ws/api/v4/spot/tickers?currency_pair=MNT_USDT"),
        ("Bybit", "https://api.bybit.com/v5/market/tickers?category=spot&symbol=MNTUSDT"),
        ("MEXC", "https://www.mexc.com/open/api/v2/market/ticker?symbol=MNT_USDT")
    ]
    
    for name, url in apis:
        try:
            async with aiohttp.ClientSession() as session:
                async with session.get(url, timeout=5) as response:
                    if response.status == 200:
                        data = await response.json()
                        
                        if name == "Gate.io":
                            if data and len(data) > 0:
                                return float(data[0]['last']), name
                        elif name == "Bybit":
                            if 'result' in data and 'list' in data['result']:
                                return float(data['result']['list'][0]['lastPrice']), name
                        elif name == "MEXC":
                            if 'data' in data and len(data['data']) > 0:
                                return float(data['data'][0]['last']), name
        except:
            continue
    
    return None, None

async def main():
    print("🤖 MNT Price Bot запускается...")
    
    if not TELEGRAM_TOKEN:
        print("❌ Ошибка: TELEGRAM_TOKEN не установлен!")
        return
    
    if not CHAT_ID:
        print("❌ Ошибка: CHAT_ID не установлен!")
        return
    
    bot = Bot(token=TELEGRAM_TOKEN)
    
    # Тестовая отправка
    try:
        await bot.send_message(
            chat_id=CHAT_ID,
            text="🟢 MNT Price Bot запущен! Буду присылать цену каждую минуту.",
            parse_mode='HTML'
        )
        print("✅ Тестовое сообщение отправлено!")
    except Exception as e:
        print(f"❌ Ошибка Telegram: {e}")
        return
    
    while True:
        try:
            price, source = await get_mnt_price()
            
            if price:
                message = f"""
💰 <b>MANTLE (MNT) PRICE</b>
┌──────────────────
│ <b>${price:,.4f}</b>
│ 🕐 {datetime.now().strftime("%H:%M:%S")}
│ 📊 {source}
└──────────────────
                """
                
                await bot.send_message(
                    chat_id=CHAT_ID,
                    text=message,
                    parse_mode='HTML'
                )
                
                print(f"✅ {datetime.now()}: ${price:.4f} с {source}")
            else:
                print(f"⚠️ {datetime.now()}: Не удалось получить цену MNT")
                await bot.send_message(
                    chat_id=CHAT_ID,
                    text="⚠️ Не удалось получить цену MNT. Пробую снова через минуту...",
                    parse_mode='HTML'
                )
                
        except Exception as e:
            print(f"❌ Ошибка: {e}")
        
        await asyncio.sleep(60)

if __name__ == "__main__":
    asyncio.run(main()) 
