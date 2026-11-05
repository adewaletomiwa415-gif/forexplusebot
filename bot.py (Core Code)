import os
import logging
import feedparser
from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes
from biquote import Biquote

logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    level=logging.INFO
)
logger = logging.getLogger(__name__)

bq = Biquote()

# Watchlist: major forex pairs
WATCH_PAIRS = ["EURUSD", "GBPUSD", "USDJPY", "AUDUSD", "USDCAD", "USDCHF", "NZDUSD"]

# Free Forex RSS sources (no API key)
RSS_FEEDS = [
    "https://www.dailyfx.com/feeds/all",
    "https://www.fxstreet.com/rss/news",
]


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "💱 **Forex Bot Ready**\n\n"
        "Commands:\n"
        "/quotes — Major pair quotes\n"
        "/calendar — This week's high-impact events\n"
        "/news — Latest Forex news\n"
        "/help — Show this menu",
        parse_mode="Markdown"
    )


async def quotes(update: Update, context: ContextTypes.DEFAULT_TYPE):
    try:
        ticks = bq.latest(WATCH_PAIRS)
        lines = ["💹 **Forex Quotes**\n"]
        for symbol, data in ticks.items():
            mid = data.get("mid", "N/A")
            change = data.get("dayDiffPercent", 0)
            arrow = "📈" if change >= 0 else "📉"
            lines.append(f"{arrow} **{symbol}**: {mid} ({change:+.2f}%)")
        await update.message.reply_text("\n".join(lines), parse_mode="Markdown")
    except Exception as e:
        logger.error(f"Quotes error: {e}")
        await update.message.reply_text("⚠️ Unable to fetch quotes. Try again later.")


async def calendar(update: Update, context: ContextTypes.DEFAULT_TYPE):
    try:
        events = bq.calendar(importance="high", countries="US,EU,GB,JP")
        if not events:
            await update.message.reply_text("No high-impact events this week.")
            return

        lines = ["📅 **High-Impact Events (This Week)**\n"]
        for ev in events[:8]:
            country = ev.get("country", "?")
            title = ev.get("title", ev.get("event", "Unknown"))
            date = ev.get("date", "")
            lines.append(f"• [{country}] {title} — {date}")
        await update.message.reply_text("\n".join(lines), parse_mode="Markdown")
    except Exception as e:
        logger.error(f"Calendar error: {e}")
        await update.message.reply_text("⚠️ Unable to fetch calendar.")


async def news(update: Update, context: ContextTypes.DEFAULT_TYPE):
    entries = []
    for url in RSS_FEEDS:
        try:
            feed = feedparser.parse(url)
            for entry in feed.entries[:3]:
                title = entry.get("title", "No title")
                link = entry.get("link", "")
                entries.append(f"• [{title}]({link})")
        except Exception as e:
            logger.warning(f"RSS failed {url}: {e}")

    if not entries:
        await update.message.reply_text("⚠️ No news available right now. RSS sources may be temporarily down.")
        return

    text = "📰 **Latest Forex News**\n\n" + "\n".join(entries[:6])
    await update.message.reply_text(text, parse_mode="Markdown", disable_web_page_preview=True)


async def help_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await start(update, context)


def main():
    token = os.environ.get("TELEGRAM_BOT_TOKEN")
    if not token:
        logger.error("TELEGRAM_BOT_TOKEN not set!")
        return

    app = Application.builder().token(token).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("help", help_cmd))
    app.add_handler(CommandHandler("quotes", quotes))
    app.add_handler(CommandHandler("calendar", calendar))
    app.add_handler(CommandHandler("news", news))

    logger.info("Forex bot starting (long polling)...")
    app.run_polling()


if __name__ == "__main__":
    main()
