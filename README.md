# -"""
Crypto Signal Bot
"""
import aiohttp
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, CallbackContext
import os

BOT_TOKEN = os.environ.get("BOT_TOKEN", "8373787175:AAEiBn6veYGg9yv6J1YdXdozOFOq1spmVr4")
BINANCE_API = "https://api.binance.com/api/v3"

COIN_MAP = {"btc": "bitcoin", "eth": "ethereum", "bnb": "binancecoin", "sol": "solana", "xrp": "ripple", "ada": "cardano", "doge": "dogecoin", "dot": "polkadot", "avax": "avalanche-2", "matic": "matic-network"}
COIN_CN = {"比特币": "btc", "以太坊": "eth", "币安币": "bnb", "索拉纳": "sol", "瑞波币": "xrp", "艾达币": "ada", "狗狗币": "doge", "波卡": "dot", "雪崩": "avax", "matic": "matic"}
PRICE_PRO = 9.99
user_lang = {}

def t(uid, key):
    lang = user_lang.get(uid, "zh")
    texts = {"zh": {"start": "📈 加密货币信号机器人\n发送文字：如比特币价格、热门", "help": "📋 支持：价格、热门、信号、订阅", "subscribe": f"⭐ VIP ${PRICE_PRO}/月", "my": "免费用户", "not_found": "未找到", "usage_price": "用法：价格 比特币", "lang_switched": "语言已切换", "unknown": "发送 帮助", "vip_more": "\n💡 VIP更多"}, "en": {"start": "📈 Crypto Signal Bot", "help": "Commands: price, top, signal", "subscribe": f"⭐ VIP ${PRICE_PRO}/month", "my": "Free", "not_found": "Not found", "usage_price": "Usage: price btc", "lang_switched": "Switched", "unknown": "Send help", "vip_more": "\n💡 VIP"}}
    return texts.get(lang, texts["zh"]).get(key, key)

def match_command(text):
    text = text.lower().strip()
    if any(k in text for k in ["价格", "价", "price", "多少"]): return "price"
    if any(k in text for k in ["热门", "排行", "top", "查看"]): return "top"
    if any(k in text for k in ["信号", "signal"]): return "signal"
    if any(k in text for k in ["订阅", "subscribe", "vip"]): return "subscribe"
    if any(k in text for k in ["帮助", "help"]): return "help"
    if any(k in text for k in ["语言", "lang"]): return "lang"
    if any(k in text for k in ["我的", "my"]): return "my"
    return None

def extract_coin(text):
    text = text.lower()
    for cn, en in COIN_CN.items():
        if cn.lower() in text: return en
    return None

async def get_price(symbol):
    coin_id = COIN_MAP.get(symbol.lower(), symbol.lower())
    try:
        async with aiohttp.ClientSession() as session:
            pair = f"{symbol.upper()}USDT"
            async with session.get(f"{BINANCE_API}/ticker/24hr", params={"symbol": pair}, timeout=aiohttp.ClientTimeout(total=5)) as resp:
                if resp.status == 200:
                    data = await resp.json()
                    return {"usd": float(data.get("lastPrice", 0)), "usd_24h_change": float(data.get("priceChangePercent", 0))}
    except: pass
    return {"usd": 0, "usd_24h_change": 0}

async def get_top_coins():
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(f"{BINANCE_API}/ticker/24hr", timeout=aiohttp.ClientTimeout(total=5)) as resp:
                if resp.status == 200:
                    data = await resp.json()
                    usdt_pairs = [x for x in data if x["symbol"].endswith("USDT")]
                    usdt_pairs.sort(key=lambda x: float(x.get("quoteVolume", 0)), reverse=True)
                    return [{"symbol": c["symbol"].replace("USDT", ""), "current_price": float(c["lastPrice"]), "price_change_percentage_24h": float(c["priceChangePercent"])} for c in usdt_pairs[:10]]
    except: pass
    return []

async def start_command(update, context): await update.message.reply_text(t(update.effective_user.id, "start"))
async def help_command(update, context): await update.message.reply_text(t(update.effective_user.id, "help"))
async def price_command(update, context):
    uid = update.effective_user.id
    text = " ".join(context.args) if context.args else ""
    status_msg = await update.message.reply_text("⏳...")
    coin = extract_coin(text) if text else None
    if not coin:
        await status_msg.edit_text(t(uid, "usage_price"))
        return
    data = await get_price(coin)
如果 data 和 data.get("usd", 0) 大于 0：
价格 = 数据["美元"]
        change = data.get("usd_24h_change", 0)
emoji = "" 如果 change > 0 否则 ""
        msg = f"💰 {COIN_MAP.get(coin, coin).upper()}\n${price:,.2f}\n24h: {emoji} {change:.1f}%"
        await status_msg.edit_text(msg)
否则：
        await status_msg.edit_text(t(uid, "not_found"))

async def top_command(update, context):
    coins = await get_top_coins()
    msg = "📊 Top 10\n\n"
    for i, c in enumerate(coins, 1):
        emoji = "🟢" if c["price_change_percentage_24h"] > 0 else "🔴"
        msg += f"{i}. {c['symbol']} ${c['current_price']:,.0f} {emoji} {c['price_change_percentage_24h']:.1f}%\n"
    await update.message.reply_text(msg)

async def signal_command(update, context):
    uid = update.effective_user.id
    coins = await get_top_coins()
消息 = “信号  ”
    for c in coins[:3]:
sig = “⚠️” 如果 c[] > 5，否则为“✅”如果 c[“price_change_percentage_24h”]
python-telegram-bot>=20.0
aiohttp>=3.9.0
网页：python main.py
python-3.11.0
