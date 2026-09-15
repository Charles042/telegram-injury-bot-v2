# telegram-injury-bot-v2
Updates of European football player injuries
import os
import logging
from datetime import time
from dotenv import load_dotenv
from telegram import Update
from telegram.ext import (
    Application,
    CommandHandler,
    ContextTypes,
)
# ============================================================
# CONFIGURATION
# ============================================================
load_dotenv()
TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN")
CHANNEL_ID = os.getenv("CHANNEL_ID")
if not TELEGRAM_TOKEN:
    raise ValueError(
        "TELEGRAM_TOKEN is missing. Add it to your .env file."
    )
if not CHANNEL_ID:
    raise ValueError(
        "CHANNEL_ID is missing. Add it to your .env file."
    )
# ============================================================
# LOGGING
# ============================================================
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    level=logging.INFO,
)
logger = logging.getLogger(__name__)
# ============================================================
# INJURY REPORT FUNCTION
# ============================================================
async def get_injury_report():
    """
    This function should obtain the latest player injury data.
    Replace the example data below with your actual data source/API.
    IMPORTANT:
    Do not invent injury information. The bot should only publish
    information retrieved from your configured source.
    """
    # --------------------------------------------------------
    # TEMPORARY TEST REPORT
    # --------------------------------------------------------
    #
    # Replace this section once we connect your actual injury
    # information source.
    #
    report = """
🏥 PLAYER INJURY UPDATE
📅 Latest Update
🇬🇧 PREMIER LEAGUE
No injury data source connected yet.
🇪🇸 LA LIGA
No injury data source connected yet.
🇮🇹 SERIE A
No injury data source connected yet.
🇩🇪 BUNDESLIGA
No injury data source connected yet.
🇫🇷 LIGUE 1
No injury data source connected yet.
🇵🇹 PRIMEIRA LIGA
No injury data source connected yet.
🇹🇷 SÜPER LIG
No injury data source connected yet.
🇧🇪 BELGIAN PRO LEAGUE
No injury data source connected yet.
━━━━━━━━━━━━━━━━━━━━
Source: Configured injury-data source
"""
    return report.strip()
# ============================================================
# SEND REPORT
# ============================================================
async def send_injury_report(
    context: ContextTypes.DEFAULT_TYPE,
    destination=None,
):
    """
    Generate and send the latest injury report.
    """
    try:
        report = await get_injury_report()
        if destination is None:
            destination = CHANNEL_ID
        # Telegram messages have a maximum length.
        # Split very long reports into smaller messages.
        max_length = 4000
        for i in range(0, len(report), max_length):
            message = report[i:i + max_length]
            await context.bot.send_message(
                chat_id=destination,
                text=message,
            )
        logger.info(
            "Injury report sent successfully to %s",
            destination,
        )
    except Exception as e:
        logger.exception("Failed to send injury report: %s", e)
# ============================================================
# /REPORT COMMAND
# ============================================================
async def report_command(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE,
):
    """
    Immediately generate and send the latest injury report
    when the user sends:
    /report
    """
    try:
        await update.message.reply_text(
            "🔄 Getting the latest player injury report..."
        )
        report = await get_injury_report()
        max_length = 4000
        for i in range(0, len(report), max_length):
            message = report[i:i + max_length]
            await update.message.reply_text(
                message
            )
    except Exception as e:
        logger.exception(
            "Error generating /report: %s",
            e,
        )
        await update.message.reply_text(
            "❌ Sorry, I couldn't generate the injury report right now."
        )
# ============================================================
# /START COMMAND
# ============================================================
async def start_command(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE,
):
    await update.message.reply_text(
        "⚽ Welcome to the Player Injury Update Bot.\n\n"
        "Use /report to get the latest injury report immediately."
    )
# ============================================================
# /HELP COMMAND
# ============================================================
async def help_command(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE,
):
    await update.message.reply_text(
        "📋 Available commands:\n\n"
        "/report - Get the latest injury report immediately\n"
        "/start - Start the bot\n"
        "/help - Show available commands\n\n"
        "Automatic reports:\n"
        "🕕 Daily - Every day at 6:00 PM\n"
        "📅 Weekly - Every Sunday at 9:00 PM"
    )
# ============================================================
# DAILY UPDATE — 6 PM
# ============================================================
async def daily_update(
    context: ContextTypes.DEFAULT_TYPE,
):
    logger.info("Running daily injury update.")
    await send_injury_report(
        context=context,
        destination=CHANNEL_ID,
    )
# ============================================================
# WEEKLY UPDATE — SUNDAY 9 PM
# ============================================================
async def weekly_update(
    context: ContextTypes.DEFAULT_TYPE,
):
    logger.info("Running Sunday weekly injury update.")
    await send_injury_report(
        context=context,
        destination=CHANNEL_ID,
    )
# ============================================================
# ERROR HANDLER
# ============================================================
async def error_handler(
    update: object,
    context: ContextTypes.DEFAULT_TYPE,
):
    logger.error(
        "Exception while processing update:",
        exc_info=context.error,
    )
# ============================================================
# MAIN
# ============================================================
def main():
    # Create Telegram application
    application = (
        Application.builder()
        .token(TELEGRAM_TOKEN)
        .build()
    )
    # --------------------------------------------------------
    # COMMAND HANDLERS
    # --------------------------------------------------------
    application.add_handler(
        CommandHandler(
            "start",
            start_command,
        )
    )
    application.add_handler(
        CommandHandler(
            "help",
            help_command,
        )
    )
    application.add_handler(
        CommandHandler(
            "report",
            report_command,
        )
    )
    # --------------------------------------------------------
    # ERROR HANDLER
    # --------------------------------------------------------
    application.add_error_handler(
        error_handler
    )
    # --------------------------------------------------------
    # DAILY 6 PM UPDATE
    # --------------------------------------------------------
    #
    # time(hour=18, minute=0)
    # means 6:00 PM.
    #
    # The bot uses the computer/server's local timezone.
    #
    application.job_queue.run_daily(
        daily_update,
        time=time(
            hour=18,
            minute=0,
        ),
        name="daily_injury_update",
    )
    # --------------------------------------------------------
    # SUNDAY 9 PM UPDATE
    # --------------------------------------------------------
    #
    # Python weekday:
    #
    # Monday    = 0
    # Tuesday   = 1
    # Wednesday = 2
    # Thursday  = 3
    # Friday    = 4
    # Saturday  = 5
    # Sunday    = 6
    #
    application.job_queue.run_daily(
        weekly_update,
        time=time(
            hour=21,
            minute=0,
        ),
        days=(6,),
        name="sunday_injury_update",
    )
    # --------------------------------------------------------
    # START BOT
    # --------------------------------------------------------
    logger.info(
        "Player Injury Telegram Bot is starting..."
    )
    application.run_polling()
# ============================================================
# RUN
# ============================================================
if __name__ == "__main__":
    main()