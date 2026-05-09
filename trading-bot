import os
import ccxt
import pandas as pd
from ta.trend import EMAIndicator, MACD
from ta.momentum import RSIIndicator
from telegram import Bot
import time

TOKEN = os.getenv("TOKEN")
CHAT_ID = os.getenv("CHAT_ID")

bot = Bot(token=TOKEN)
exchange = ccxt.binance()

pairs = ["BTC/USDT", "ETH/USDT"]

def get_data(pair, tf):
    ohlcv = exchange.fetch_ohlcv(pair, tf, limit=200)
    return pd.DataFrame(ohlcv, columns=['time','open','high','low','close','volume'])

def get_trend(pair):
    df = get_data(pair, '1h')
    df['ema200'] = EMAIndicator(df['close'], 200).ema_indicator()
    return "BUY" if df.iloc[-1]['close'] > df.iloc[-1]['ema200'] else "SELL"

def get_entry(pair):
    df = get_data(pair, '15m')

    df['ema50'] = EMAIndicator(df['close'], 50).ema_indicator()
    df['ema200'] = EMAIndicator(df['close'], 200).ema_indicator()
    df['rsi'] = RSIIndicator(df['close'], 14).rsi()

    macd = MACD(df['close'])
    df['macd'] = macd.macd()
    df['signal'] = macd.macd_signal()

    last = df.iloc[-1]

    if last['ema50'] > last['ema200'] and last['macd'] > last['signal'] and 40 < last['rsi'] < 65:
        return "BUY", last['close'], df

    if last['ema50'] < last['ema200'] and last['macd'] < last['signal'] and 35 < last['rsi'] < 60:
        return "SELL", last['close'], df

    return None, None, None

def calc_sl_tp(signal, entry, df):
    if signal == "BUY":
        sl = df['low'].rolling(10).min().iloc[-1]
        tp = entry + (entry - sl) * 2
    else:
        sl = df['high'].rolling(10).max().iloc[-1]
        tp = entry - (sl - entry) * 2
    return sl, tp

while True:
    for pair in pairs:
        trend = get_trend(pair)
        signal, entry, df = get_entry(pair)

        if signal and signal == trend:
            sl, tp = calc_sl_tp(signal, entry, df)

            msg = f"""
🚀 SIGNAL

{pair}
Type: {signal}

Entry: {round(entry, 4)}
SL: {round(sl, 4)}
TP: {round(tp, 4)}
"""

            bot.send_message(chat_id=CHAT_ID, text=msg)

    time.sleep(60)
