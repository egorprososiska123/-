from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

TOKEN = "ВАШ_ТОКЕН_БОТА"

# словарь: user_id -> коэффициент
user_targets = {}

# коэффициент по умолчанию
DEFAULT_TARGET = 1.9487784992

def get_user_target(user_id: int) -> float:
    """Получить коэффициент для пользователя (или дефолтный)."""
    return user_targets.get(user_id, DEFAULT_TARGET)

def set_user_target(user_id: int, value: float):
    """Установить коэффициент для пользователя."""
    user_targets[user_id] = value

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    target = get_user_target(user_id)

    await update.message.reply_text(
        "Привет! Введи первое число (делитель), а я подберу второе так, "
        "чтобы при делении получилось ≈ твой коэффициент.\n"
        f"Сейчас коэффициент = {target}\n\n"
        "Измени его командой: /set <число>\n"
        "Показать текущий коэффициент: /get"
    )

async def set_target(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    try:
        if not context.args:
            await update.message.reply_text("❌ Нужно ввести число. Пример: /set 1.9423")
            return

        new_value = float(context.args[0])
        set_user_target(user_id, new_value)
        await update.message.reply_text(f"✅ Новый коэффициент установлен: {new_value}")

    except ValueError:
        await update.message.reply_text("❌ Ошибка! Нужно ввести число.")

async def get_target(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    target = get_user_target(user_id)
    await update.message.reply_text(f"📌 Текущий коэффициент: {target}")

async def handle_number(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    target = get_user_target(user_id)

    try:
        num1 = float(update.message.text)

        # считаем второе число
        num2 = num1 * target
        num2_rounded = round(num2)  # округлим до ближайшего целого

        ratio = num2_rounded / num1

        msg = (
            f"Для числа {num1} подобрал второе:\n"
            f"{num2_rounded}\n\n"
            f"Проверка: {num2_rounded} / {num1} = {ratio:.10f}\n"
            f"(текущий коэффициент: {target})"
        )

        await update.message.reply_text(msg)

    except ValueError:
        await update.message.reply_text("Нужно ввести число!")

def main():
    app = Application.builder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("set", set_target))
    app.add_handler(CommandHandler("get", get_target))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_number))

    app.run_polling()

if __name__ == "__main__":
    main()
