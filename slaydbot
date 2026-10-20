import asyncio
import logging
from io import BytesIO

from aiogram import Bot, Dispatcher, Router, F, types
from aiogram.filters import Command
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.storage.memory import MemoryStorage
from aiogram.types import (
    InlineKeyboardButton,
    InlineKeyboardMarkup,
    KeyboardButton,
    ReplyKeyboardMarkup,
    ReplyKeyboardRemove
)
from PIL import Image, ImageDraw, ImageFont
from reportlab.lib.pagesizes import A4
from reportlab.pdfgen import canvas

# -------------------------------------------------------------
# CONFIGURATION
# -------------------------------------------------------------
BOT_TOKEN = "8626040144:AAGaWrLcXR3KlAJe0bxclJyMxbYQgDxEVlo"  # Bot tokenini kiriting
ADMIN_ID = 8052376816  # Admin Telegram ID-sini kiriting (masalan: 123456789)

# Majburiy kanallar ro'yxati (username yoki -100 ID)
REQUIRED_CHANNELS = ["@norboyevbuilder"]  # O'zingizning kanalingizni kiriting

CARD_NUMBER = "4073 4200 7949 1878"
CARD_HOLDER = "Norboyev Nazarbek"

# Dizayn uslublari ro'yxati
SLIDE_DESIGNS = {
    "1": "Minimalist (Sodda va toza)",
    "2": "IT / Tech (Zamonaviy va texnologik)",
    "3": "Biznes & Startap (Taqdimotlar uchun)",
    "4": "Kreativ & Art (Yorqin va nojo'xot)",
    "5": "Ilmiy / Akademik (Universitet va maktab)",
    "6": "Dark Mode (Qorong'u uslub)",
    "7": "Cyberpunk (Futuristik)",
    "8": "Elegant & Gold (Hashamatli va klassik)",
    "9": "Nature / Eco (Tabiiy va yashil va pastel)",
    "10": "Infografik (Raqamlar va grafiklar ko'p)"
}

# Narxlar xaritasi
PRICES = {
    "pages_10": {"pages": 10, "price": 2000},
    "pages_15": {"pages": 15, "price": 5000},
    "pages_20": {"pages": 20, "price": 7000},
}


# -------------------------------------------------------------
# FSM (STATES)
# -------------------------------------------------------------
class OrderSlide(StatesGroup):
    waiting_for_topic = State()
    waiting_for_package = State()
    waiting_for_payment = State()
    waiting_for_design = State()


class ReferatState(StatesGroup):
    waiting_for_topic = State()


class TestState(StatesGroup):
    waiting_for_topic = State()


class DiplomState(StatesGroup):
    waiting_for_topic = State()


class MustaqilIshState(StatesGroup):
    waiting_for_topic = State()


# -------------------------------------------------------------
# ROUTER & BOT SETUP
# -------------------------------------------------------------
router = Router()
logging.basicConfig(level=logging.INFO)


# -------------------------------------------------------------
# HELPER FUNCTIONS
# -------------------------------------------------------------
async def check_subscription(bot: Bot, user_id: int) -> bool:
    """Foydalanuvchi majburiy kanallarga a'zo ekanligini tekshiradi."""
    for channel in REQUIRED_CHANNELS:
        try:
            member = await bot.get_chat_member(chat_id=channel, user_id=user_id)
            if member.status in ["left", "kicked"]:
                return False
        except Exception as e:
            logging.error(f"Subscription check error for {channel}: {e}")
            return False
    return True


def get_sub_keyboard():
    """Obuna bo'lish tugmalarini yasash."""
    buttons = []
    for ch in REQUIRED_CHANNELS:
        url = f"https://t.me/{ch.replace('@', '')}"
        buttons.append([InlineKeyboardButton(text=f"📢 {ch} ga obuna bo'lish", url=url)])
    buttons.append([InlineKeyboardButton(text="🔄 Tekshirish", callback_data="check_sub")])
    return InlineKeyboardMarkup(inline_keyboard=buttons)


def get_main_menu():
    """Asosiy menyu klaviaturasini qaytaradi."""
    return ReplyKeyboardMarkup(
        keyboard=[
            [
                KeyboardButton(text="📘 Slayd"),
                KeyboardButton(text="📚 Referat"),
                KeyboardButton(text="🧪 Test")
            ],
            [
                KeyboardButton(text="🎓 Diplom ishi"),
                KeyboardButton(text="📝 Mustaqil ish")
            ],
            [
                KeyboardButton(text="📑 Referat yaratish"),
                KeyboardButton(text="✅ Test / Quiz")
            ],
            [KeyboardButton(text="👤 Profil")]
        ],
        resize_keyboard=True,
        selective=True,
        input_field_placeholder="Tanlang..."
    )


def get_design_palette(design_id: str):
    palettes = {
        "1": ("#F8FAFC", "#0F172A", "#2563EB", "#94A3B8"),
        "2": ("#0F172A", "#E2E8F0", "#22D3EE", "#38BDF8"),
        "3": ("#F8FAFC", "#111827", "#14B8A6", "#2DD4BF"),
        "4": ("#FFF7ED", "#7C2D12", "#FB7185", "#F59E0B"),
        "5": ("#F0FDF4", "#14532D", "#10B981", "#84CC16"),
        "6": ("#0B1120", "#E2E8F0", "#7C3AED", "#8B5CF6"),
        "7": ("#050816", "#E2E8F0", "#00F5D4", "#FF006E"),
        "8": ("#1F2937", "#F8E7B5", "#D4AF37", "#FDE68A"),
        "9": ("#ECFDF5", "#14532D", "#22C55E", "#A7F3D0"),
        "10": ("#F8FAFC", "#0F172A", "#3B82F6", "#60A5FA")
    }
    return palettes.get(design_id, palettes["1"])


def build_slide_plan(topic: str, pages: int, design_name: str):
    topic_clean = topic.strip() or "Mavzu"
    topic_key = topic_clean[:90]

    slides = [
        {
            "title": "Muammo va sabablar",
            "subtitle": "Aqly AI • Final taqdimot",
            "bullets": [
                f"{topic_key} mavzusining dolzarbligi va bugungi kundagi ahamiyati aniq ko'rsatiladi.",
                "Asosiy maqsad, muammoning qo'yilishi va tadqiqotning asosiy yo'nalishlari aniq bayon qilinadi.",
                "Tanqidiy fikr, nazariy asos va amaliy jihatlar bir ma'noda bog'lanadi.",
                "Mavzuga oid mavjud muammolar, cheklovlar va imkoniyatlar qiyosiy tarzda tahlil qilinadi.",
                "Taqdimotning asosiy maqsadi auditoriyani ma'lumot bilan tanishtirish va samarali yechimlarni tushuntirishdan iborat."
            ],
            "stats": [
                ("Natija", "Aniq va tushunarli"),
                ("Auditoriya", "Talabalar / ishtirokchilar"),
                ("Maqsad", "Analiz va taqdimot")
            ],
            "diagram": "Maqsad → Muammo → Yechim → Natija",
            "mini_text": "Mavzu dolzarbligi, muammo va maqsad qisqa, ravshan tarzda yoritiladi.",
            "cover": True
        },
        {
            "title": "Muammo va sabablar",
            "subtitle": "Tahliliy ko'rinish",
            "bullets": [
                f"{topic_key} bo'yicha mavjud muammolarni aniqlash va sabablarini ajratish muhim hisoblanadi.",
                "Mavzu bo'yicha mavjud yondashuvlar va ularning kamchiliklari batafsil ko'rsatiladi.",
                "Kundalik hayot, ta'lim yoki biznesdagi ta'siri misollar bilan tasvirlanadi.",
                "Agar muammo yechimga bog'liq bo'lsa, uning oqibati va natijalarini hisobga olish kerak.",
                "Yechimni samarali qilish uchun mavjud cheklovlar va imkoniyatlar solishtiriladi."
            ],
            "stats": [
                ("Sabab 1", "Mavjud yondashuvlar yetarli emas"),
                ("Sabab 2", "Amaliy ko'rsatkichlar kam"),
                ("Sabab 3", "Tahlil va tushuntirish zarur")
            ],
            "diagram": "Muammo → Sabab → Oqibat → Yechim",
            "mini_text": "Asosiy omillar, oqibatlar va yechimga qadamlar aniq ifodalangan."
        }
    ]

    for i in range(3, pages + 1):
        slides.append(
            {
                "title": f"Slayd {i}",
                "subtitle": f"{i - 2}-bosqich / asosiy qism",
                "bullets": [
                    f"{topic_key} bo'yicha nazariy va amaliy tahlillar keng yoritiladi.",
                    "Asosiy fikrlar, misollar va muqobil yondashuvlar taqdim etiladi.",
                    "Samaradorlik, foyda, xavfsizlik va natija ko'rsatkichlari taqqoslanib tahlil qilinadi.",
                    "Maqsadga muvofiq yechimlar, ustunliklar va ko'rsatkichlar aniq yoritiladi.",
                    "Natijalarni tasdiqlovchi real misollar va amaliy tajriba keltiriladi."
                ],
                "stats": [
                    ("Fikr", f"{i} ta muhim nuqta"),
                    ("Tekshiruv", "Amaliy va ilmiy"),
                    ("Natija", "Aniq va ko'rsatilgan")
                ],
                "diagram": f"Bosqich {i - 1} → Tavsif → Natija → Xulosa",
                "mini_text": f"Ushbu slaydda mantiqiy ketma-ketlik, tajriba va tahlil natijalari bayon qilinadi."
            }
        )

    slides.append(
        {
            "title": "Xulosa va tavsiyalar",
            "subtitle": "Umumiy natija",
            "bullets": [
                "Asosiy xulosalar qisqa, aniq va mantiqiy tarzda jamlanadi.",
                "Kelajakdagi istiqbollar, tavsiyalar va amaliy yechimlar ishlab chiqiladi.",
                "Mavzuga oid qo'shimcha yondashuvlar va istiqbolli imkoniyatlar yoritiladi.",
                "Taqdimot oxirida auditoriya uchun muhim fikrlar birlashtirilib, xulosa shakllantiriladi.",
                "So'nggi qismda eng muhim fikrlar, real javoblar va keyingi harakatlar bitta xulosada jamlanadi."
            ],
            "stats": [
                ("Xulosa", "Keng va tushunarli"),
                ("Tavsiyalar", "Amaliy va aniq"),
                ("E'tibor", "O'quvchi / auditoriya")
            ],
            "diagram": "Xulosa → Tavsiya → Amalga oshirish",
            "mini_text": "Final qismda eng muhim fikrlar va keyingi qadamlari bir butun xulosa sifatida beriladi."
        }
    )

    return slides


def generate_slide_preview(topic: str, design_name: str, pages: int, design_id: str):
    bg, text_color, accent, secondary = get_design_palette(design_id)
    width, height = 1920, 1080
    img = Image.new("RGB", (width, height), bg)
    draw = ImageDraw.Draw(img)

    try:
        font_title = ImageFont.truetype("arialbd.ttf", 42)
        font_sub = ImageFont.truetype("arial.ttf", 24)
        font_body = ImageFont.truetype("arial.ttf", 22)
        font_small = ImageFont.truetype("arial.ttf", 17)
        font_card = ImageFont.truetype("arial.ttf", 19)
    except Exception:
        font_title = ImageFont.load_default()
        font_sub = ImageFont.load_default()
        font_body = ImageFont.load_default()
        font_small = ImageFont.load_default()
        font_card = ImageFont.load_default()

    draw.rectangle((0, 0, width, 72), fill=accent)
    draw.text((24, 18), f"Aqly AI — {design_name}", font=font_sub, fill="white")
    draw.text((1800, 20), f"1/{pages}", font=font_small, fill="white")

    left_x = 70
    left_y = 120
    left_w = 1180
    left_h = 820
    draw.rounded_rectangle((left_x, left_y, left_x + left_w, left_y + left_h), radius=28, fill=(246, 248, 246),
                           outline=(80, 80, 80), width=2)

    title_text = topic[:58] + ("..." if len(topic) > 58 else "")
    draw.text((left_x + 42, left_y + 28), title_text, font=font_title, fill=text_color)
    draw.text((left_x + 42, left_y + 84), "Aqly AI • Final taqdimot", font=font_small, fill=(120, 120, 120))

    bullet_items = [
        "Mavzuning dolzarbligi va bugungi kundagi ahamiyati aniq ko'rsatiladi.",
        "Asosiy maqsad, muammoning qo'yilishi va tadqiqotning asosiy yo'nalishlari bayon qilinadi.",
        "Tanqidiy fikr, nazariy asos va amaliy jihatlar bir ma'noda bog'lanadi.",
        "Tadqiqot natijalari, talabga muvofiqligi va jamiyatga ta'siri yoritiladi.",
        "Taqdimotning asosiy maqsadi auditoriyani ma'lumot bilan tanishtirish va samarali yechimlarni tushuntirishdan iborat."
    ]
    bullet_x = left_x + 52
    bullet_y = left_y + 152
    for item in bullet_items:
        draw.text((bullet_x + 16, bullet_y), "•", font=font_body, fill=accent)
        draw.text((bullet_x + 38, bullet_y), item, font=font_body, fill=(70, 70, 70))
        bullet_y += 50

    right_x = 1335
    right_y = 172
    card_w = 470
    card_h = 120
    card_gap = 18
    info_cards = [
        ("Natija", "Aniq va tushunarli"),
        ("Auditoriya", "Talabalar / ijtimoiy guruhlar"),
        ("Maqsad", "Analiz va taqdimot")
    ]

    for title, value in info_cards:
        draw.rounded_rectangle((right_x, right_y, right_x + card_w, right_y + card_h), radius=16, fill=(188, 204, 196),
                               outline=(180, 195, 187), width=1)
        draw.text((right_x + 24, right_y + 20), title, font=font_card, fill=(55, 55, 55))
        draw.text((right_x + 24, right_y + 58), value, font=font_small, fill=(75, 75, 75))
        right_y += card_h + card_gap

    draw.rounded_rectangle((right_x, right_y, right_x + card_w, right_y + 166), radius=18, fill=accent)
    draw.text((right_x + 24, right_y + 20), "Analiz", font=font_card, fill="white")
    draw.text((right_x + 24, right_y + 58), "Maqsad", font=font_small, fill="white")
    draw.text((right_x + 24, right_y + 82), "Muammo", font=font_small, fill="white")
    draw.text((right_x + 24, right_y + 106), "Yechim", font=font_small, fill="white")

    draw.rectangle((0, 1014, width, 1080), fill=(0, 0, 0))
    draw.rounded_rectangle((0, 1000, 330, 1080), radius=8, fill=accent)
    draw.text((28, 1018), "Aqly AI", font=font_small, fill="white")
    draw.text((28, 1042), "Mavzu: " + topic[:26], font=font_small, fill=(220, 220, 220))

    buffer = BytesIO()
    img.save(buffer, format="PNG")
    return buffer.getvalue()


def wrap_text_for_pdf(text: str, max_chars: int):
    words = text.split()
    lines = []
    current = ""
    for word in words:
        candidate = f"{current} {word}".strip()
        if len(candidate) <= max_chars:
            current = candidate
        else:
            if current:
                lines.append(current)
            current = word
    if current:
        lines.append(current)
    return lines


def generate_slide_pdf(topic: str, design_name: str, pages: int, design_id: str):
    slides = build_slide_plan(topic, pages, design_name)
    buffer = BytesIO()
    pdf = canvas.Canvas(buffer, pagesize=(1920, 1080))
    width, height = 1920, 1080

    bg, text_color, accent, secondary = get_design_palette(design_id)
    body_fill = (1, 1, 1) if bg in ["#0B1120", "#050816"] else (0.96, 0.98, 0.98)

    for i, slide in enumerate(slides):
        pdf.setFillColorRGB(*hex_to_rgb(bg))
        pdf.rect(0, 0, width, height, fill=1, stroke=0)

        pdf.setFillColorRGB(*hex_to_rgb(accent))
        pdf.rect(0, 0, width, 72, fill=1, stroke=0)
        pdf.setFillColorRGB(1, 1, 1)
        pdf.setFont("Helvetica-Bold", 20)
        pdf.drawString(28, 26, f"Aqly AI — {design_name}")
        pdf.drawRightString(width - 28, 26, f"{i + 1}/{len(slides)}")

        left_x = 70
        left_y = 120
        left_w = 1180
        left_h = 820
        pdf.setFillColorRGB(*body_fill)
        pdf.roundRect(left_x, left_y, left_w, left_h, 28, fill=1, stroke=1)

        pdf.setFillColorRGB(*hex_to_rgb(text_color))
        pdf.setFont("Helvetica-Bold", 26)
        title = slide["title"]
        title_lines = wrap_text_for_pdf(title, 28)
        for idx, line in enumerate(title_lines[:2]):
            pdf.drawString(left_x + 42, left_y + left_h - 52 - (idx * 30), line)

        pdf.setFont("Helvetica", 12)
        pdf.setFillColorRGB(*hex_to_rgb(accent))
        pdf.drawString(left_x + 42, left_y + left_h - 96, "Aqly AI • Final taqdimot")

        pdf.setFillColorRGB(*hex_to_rgb(text_color))
        pdf.setFont("Helvetica", 18)
        bullet_y = left_y + left_h - 170
        for bullet in slide["bullets"][:5]:
            if bullet_y < 140:
                break
            pdf.drawString(left_x + 52, bullet_y, "•")
            lines = wrap_text_for_pdf(bullet, 58)
            for line in lines[:2]:
                pdf.drawString(left_x + 82, bullet_y, line)
                bullet_y -= 26
            bullet_y -= 8

        right_x = 1335
        right_y = 190
        right_w = 470
        card_h = 118
        card_gap = 24
        for label, value in slide["stats"]:
            pdf.setFillColorRGB(*hex_to_rgb(secondary))
            pdf.roundRect(right_x, right_y, right_w, card_h, 18, fill=1, stroke=0)
            pdf.setFillColorRGB(*hex_to_rgb(text_color))
            pdf.setFont("Helvetica-Bold", 18)
            pdf.drawString(right_x + 26, right_y + 64, label)
            pdf.setFont("Helvetica", 11)
            pdf.drawString(right_x + 26, right_y + 38, value[:28])
            right_y += card_h + card_gap

        pdf.setFillColorRGB(*hex_to_rgb(accent))
        pdf.roundRect(right_x, right_y, right_w, 170, 20, fill=1, stroke=0)
        pdf.setFillColorRGB(1, 1, 1)
        pdf.setFont("Helvetica-Bold", 18)
        pdf.drawString(right_x + 26, right_y + 120, "Analiz")
        pdf.setFont("Helvetica", 12)
        pdf.drawString(right_x + 26, right_y + 82, "Maqsad")
        pdf.drawString(right_x + 26, right_y + 58, "Muammo")
        pdf.drawString(right_x + 26, right_y + 34, "Yechim")

        pdf.setFillColorRGB(*hex_to_rgb(accent))
        pdf.roundRect(0, 1000, 330, 80, 8, fill=1, stroke=0)
        pdf.setFillColorRGB(1, 1, 1)
        pdf.setFont("Helvetica-Bold", 12)
        pdf.drawString(24, 1030, "Aqly AI")
        pdf.setFont("Helvetica", 8)
        pdf.drawString(24, 1010, f"Mavzu: {topic[:28]}")

        if i < len(slides) - 1:
            pdf.showPage()

    pdf.save()
    return buffer.getvalue()


def hex_to_rgb(value: str):
    value = value.lstrip('#')
    if len(value) == 3:
        value = ''.join(ch * 2 for ch in value)
    return tuple(int(value[i:i + 2], 16) / 255 for i in range(0, 6, 2))


# -------------------------------------------------------------
# HANDLERS
# -------------------------------------------------------------

# --- 1. /start & Majburiy obuna ---
@router.message(Command("start"))
async def cmd_start(message: types.Message, bot: Bot, state: FSMContext):
    await state.clear()
    is_subbed = await check_subscription(bot, message.from_user.id)
    if not is_subbed:
        await message.answer(
            "⚠️ **Botdan foydalanish uchun quyidagi kanallarga obuna bo'ling:**",
            reply_markup=get_sub_keyboard(),
            parse_mode="Markdown"
        )
    else:
        await message.answer(
            "Salom! Men Aqly — AI yordamchi botman.\n\nQuyidagilardan birini tanlang:",
            reply_markup=get_main_menu()
        )


@router.callback_query(F.data == "check_sub")
async def cb_check_sub(callback: types.CallbackQuery, bot: Bot):
    is_subbed = await check_subscription(bot, callback.from_user.id)
    if is_subbed:
        await callback.message.delete()
        await callback.message.answer(
            "✅ Obuna tasdiqlandi! Asosiy menyu:",
            reply_markup=get_main_menu()
        )
    else:
        await callback.answer("❌ Hali barcha kanallarga obuna bo'lmadingiz!", show_alert=True)


# --- 2. Slayd bo'limini boshlash ---
@router.message(F.text == "📘 Slayd")
async def start_slide_order(message: types.Message, bot: Bot, state: FSMContext):
    if not await check_subscription(bot, message.from_user.id):
        await message.answer("⚠️ Avval kanallarga obuna bo'ling!", reply_markup=get_sub_keyboard())
        return

    await state.set_state(OrderSlide.waiting_for_topic)
    await message.answer(
        "✏️ **Slayd mavzusini kiriting:**\n\n*(Masalan: Sun'iy intellektning kelajagi va jamiyatga ta'siri)*",
        reply_markup=ReplyKeyboardRemove(),
        parse_mode="Markdown"
    )


# --- 3. Mavzu kiritilgandan so'ng narxlar menyusi ---
@router.message(OrderSlide.waiting_for_topic)
async def process_topic(message: types.Message, state: FSMContext):
    topic = message.text.strip()
    await state.update_data(topic=topic)
    await state.set_state(OrderSlide.waiting_for_package)

    kb = InlineKeyboardMarkup(
        inline_keyboard=[
            [InlineKeyboardButton(text="🔹 10 ta varoq — 2 000 so'm", callback_data="pages_10")],
            [InlineKeyboardButton(text="🔹 15 ta varoq — 5 000 so'm", callback_data="pages_15")],
            [InlineKeyboardButton(text="🔹 20 ta varoq — 7 000 so'm", callback_data="pages_20")],
        ]
    )

    await message.answer(
        f"📌 **Mavzu:** {topic}\n\n👇 **Varoqlar soni va paketni tanlang:**",
        reply_markup=kb,
        parse_mode="Markdown"
    )


# --- 4. Paket tanlangandan so'ng to'lov ma'lumotlarini berish ---
@router.callback_query(OrderSlide.waiting_for_package, F.data.startswith("pages_"))
async def process_package(callback: types.CallbackQuery, state: FSMContext):
    package_key = callback.data
    package_info = PRICES[package_key]

    await state.update_data(
        pages=package_info["pages"],
        price=package_info["price"]
    )
    await state.set_state(OrderSlide.waiting_for_payment)

    text = (
        f"💳 **To'lov ma'lumotlari:**\n\n"
        f"📱 **Karta:** `{CARD_NUMBER}`\n"
        f"👤 **Egasi:** {CARD_HOLDER}\n"
        f"💰 **To'lov summasi:** {package_info['price']} so'm\n"
        f"📄 **Varoqlar soni:** {package_info['pages']} ta\n\n"
        f"To'lovni amalga oshirgach, pastdagi **'✅ To'lov qildim'** tugmasini bosing."
    )

    kb = InlineKeyboardMarkup(
        inline_keyboard=[
            [InlineKeyboardButton(text="✅ To'lov qildim", callback_data="paid_confirm")]
        ]
    )

    await callback.message.edit_text(text, reply_markup=kb, parse_mode="Markdown")


# --- 5. Foydalanuvchi "To'lov qildim" deb tasdiqlashi ---
@router.callback_query(OrderSlide.waiting_for_payment, F.data == "paid_confirm")
async def process_paid_confirm(callback: types.CallbackQuery, bot: Bot, state: FSMContext):
    data = await state.get_data()
    user = callback.from_user

    await callback.message.edit_text(
        "⏳ **To'lovingiz admin tomonidan tekshirilmoqda.**\nTasdiqlanishi bilan xabar beramiz!",
        parse_mode="Markdown"
    )

    # Adminga xabar yuborish
    admin_text = (
        f"💳 **Yangi to'lov!**\n\n"
        f"👤 **Foydalanuvchi:** {user.full_name}\n"
        f"🆔 **ID:** `{user.id}`\n"
        f"🔗 **Username:** @{user.username if user.username else 'Yo\'q'}\n"
        f"📌 **Mavzu:** {data.get('topic')}\n"
        f"📄 **Varoqlar:** {data.get('pages')} ta\n"
        f"💰 **Summa:** {data.get('price')} so'm"
    )

    admin_kb = InlineKeyboardMarkup(
        inline_keyboard=[
            [
                InlineKeyboardButton(text="✅ Tushdi", callback_data=f"admin_pay_yes:{user.id}"),
                InlineKeyboardButton(text="❌ Tushmadi", callback_data=f"admin_pay_no:{user.id}")
            ]
        ]
    )

    try:
        await bot.send_message(chat_id=ADMIN_ID, text=admin_text, reply_markup=admin_kb, parse_mode="Markdown")
    except Exception as e:
        logging.error(f"Adminga xabar yuborishda xatolik: {e}")


# --- 6. Admin Javobi (✅ Tushdi / ❌ Tushmadi) ---
@router.callback_query(F.data.startswith("admin_pay_"))
async def process_admin_decision(callback: types.CallbackQuery, bot: Bot, state: FSMContext):
    payload = callback.data
    is_approved = payload.startswith("admin_pay_yes")
    user_id = int(payload.split(":", 1)[1])

    if is_approved:
        await callback.message.edit_text(callback.message.text + "\n\n✅ **To'lov tasdiqlandi.**")

        # Dizayn tanlash menyusi
        designs_text = "🎨 **Slayd dizaynini tanlang (1-10):**\n\n"
        buttons = []
        row = []
        for key, name in SLIDE_DESIGNS.items():
            designs_text += f"**{key}.** {name}\n"
            row.append(InlineKeyboardButton(text=f"{key}", callback_data=f"select_design:{key}"))
            if len(row) == 5:
                buttons.append(row)
                row = []
        if row:
            buttons.append(row)

        design_kb = InlineKeyboardMarkup(inline_keyboard=buttons)

        # Foydalanuvchiga xabar
        try:
            await bot.send_message(
                chat_id=user_id,
                text=f"🎉 **To'lovingiz tasdiqlandi!**\n\n{designs_text}\nKerakli dizayn raqamini tanlang:",
                reply_markup=design_kb,
                parse_mode="Markdown"
            )
        except Exception as e:
            logging.error(f"Foydalanuvchiga xabar yuborishda xatolik: {e}")
    else:
        await callback.message.edit_text(callback.message.text + "\n\n❌ **To'lov rad etildi.**")
        try:
            await bot.send_message(
                chat_id=user_id,
                text="❌ **To'lov tasdiqlanmadi.** Iltimos, administrator bilan bog'laning yoki qayta urinib ko'ring.",
                reply_markup=get_main_menu()
            )
        except Exception as e:
            logging.error(f"Foydalanuvchiga xabar yuborishda xatolik: {e}")


# --- 7. Dizayn tanlash va Gamma-style Reja Generatsiyasi ---
@router.callback_query(F.data.startswith("select_design:"))
async def process_design_selection(callback: types.CallbackQuery, state: FSMContext):
    design_id = callback.data.split(":")[1]
    design_name = SLIDE_DESIGNS.get(design_id, "Standard")

    data = await state.get_data()
    topic = data.get("topic", "Mavzu ko'rsatilmadi")
    pages = data.get("pages", 10)

    await callback.message.edit_text(
        "⚡️ **Gamma AI uslubida slayd rejasi va tarkibi tayyorlanmoqda...**\n*(Iltimos, kuting)*",
        parse_mode="Markdown")

    preview_bytes = generate_slide_preview(topic, design_name, pages, design_id)
    pdf_bytes = generate_slide_pdf(topic, design_name, pages, design_id)

    await callback.message.answer_photo(
        photo=types.BufferedInputFile(preview_bytes, filename=f"slide_preview_{design_id}.png"),
        caption=(
            f"🎨 **Slayd dizayni tayyor!**\n\n"
            f"📌 Mavzu: {topic}\n"
            f"🎯 Dizayn: {design_name}\n"
            f"📄 Varoqlar: {pages} ta"
        ),
        parse_mode="Markdown"
    )

    await callback.message.answer_document(
        document=types.BufferedInputFile(pdf_bytes, filename=f"{topic[:20].replace(' ', '_')}_slayd.pdf"),
        caption="📄 **To'liq ma'lumotlar bilan PDF fayl tayyor!**",
        parse_mode="Markdown"
    )

    await callback.message.answer(
        "✅ Slaydlar tayyor bo'ldi. Keyingi buyurtma uchun yana tanlang.",
        reply_markup=get_main_menu()
    )
    await state.clear()


# =============================================================
# QO'SHILGAN YANGI BO'LIMLAR (Referat, Test, Diplom, Mustaqil ish, Profil)
# =============================================================

# --- 📚 Referat va 📑 Referat yaratish ---
@router.message(F.text.in_({"📚 Referat", "📑 Referat yaratish"}))
async def cmd_referat(message: types.Message, bot: Bot, state: FSMContext):
    if not await check_subscription(bot, message.from_user.id):
        await message.answer("⚠️ Avval kanallarga obuna bo'ling!", reply_markup=get_sub_keyboard())
        return
    await state.set_state(ReferatState.waiting_for_topic)
    await message.answer(
        "📚 **Referat mavzusini kiriting:**\n\n*(Bot siz uchun reja, kirish, asosiy qism va xulosa qismlarini tayyorlab beradi)*",
        reply_markup=ReplyKeyboardRemove(),
        parse_mode="Markdown"
    )


@router.message(ReferatState.waiting_for_topic)
async def process_referat_topic(message: types.Message, state: FSMContext):
    topic = message.text.strip()
    await state.clear()

    referat_text = (
        f"📚 **REFERAT**\n\n"
        f"**Mavzu:** {topic}\n\n"
        f"**REJA:**\n"
        f"1. Kirish\n"
        f"2. {topic}ning nazariy asoslari\n"
        f"3. Asosiy tahlillar va amaliyot\n"
        f"4. Xulosa\n"
        f"5. Foydalanilgan adabiyotlar\n\n"
        f"--- \n"
        f"**1. KIRISH**\n"
        f"Mazkur {topic} mavzusi bugungi kunda dolzarb masalalardan biri bo'lib, uning nazariy va amaliy ahamiyati juda yuqori...\n\n"
        f"**2. ASOSIY QISM**\n"
        f"Ushbu yo'nalishda olib borilgan tadqiqotlar shuni ko'rsatadiki, {topic} bo'yicha qator yechimlar mavjud...\n\n"
        f"**3. XULOSA**\n"
        f"Xulosa o'rnida aytish mumkinki, {topic} masalasini chuqur o'rganish muhim natijalarga olib keladi."
    )

    await message.answer(referat_text, reply_markup=get_main_menu(), parse_mode="Markdown")


# --- 🧪 Test va ✅ Test / Quiz ---
@router.message(F.text.in_({"🧪 Test", "✅ Test / Quiz"}))
async def cmd_test(message: types.Message, bot: Bot, state: FSMContext):
    if not await check_subscription(bot, message.from_user.id):
        await message.answer("⚠️ Avval kanallarga obuna bo'ling!", reply_markup=get_sub_keyboard())
        return
    await state.set_state(TestState.waiting_for_topic)
    await message.answer(
        "🧪 **Test yoki fan yo'nalishini kiriting:**\n\n*(Masalan: Tarix, Matematika yoki Dasturlash asoslari)*",
        reply_markup=ReplyKeyboardRemove(),
        parse_mode="Markdown"
    )


@router.message(TestState.waiting_for_topic)
async def process_test_topic(message: types.Message, state: FSMContext):
    topic = message.text.strip()
    await state.clear()

    test_text = (
        f"🧪 **'{topic}' bo'yicha test savollari:**\n\n"
        f"**1-savol:** {topic} tushunchasining asosiy maqsadi nima?\n"
        f"A) Ma'lumotlarni o'rganish\n"
        f"B) Xatoliklarni bartaraf etish\n"
        f"C) Tizimli yondashuvni shakllantirish\n"
        f"D) Barchasi to'g'ri\n"
        f"✅ *To'g'ri javob: D*\n\n"
        f"**2-savol:** Ushbu yo'nalish qaysi sohaga tegishli?\n"
        f"A) Ilmiy-amaliy\n"
        f"B) Faqat nazariy\n"
        f"C) Texnologik\n"
        f"D) Iqtisodiy\n"
        f"✅ *To'g'ri javob: A*"
    )

    await message.answer(test_text, reply_markup=get_main_menu(), parse_mode="Markdown")


# --- 🎓 Diplom ishi ---
@router.message(F.text == "🎓 Diplom ishi")
async def cmd_diplom(message: types.Message, bot: Bot, state: FSMContext):
    if not await check_subscription(bot, message.from_user.id):
        await message.answer("⚠️ Avval kanallarga obuna bo'ling!", reply_markup=get_sub_keyboard())
        return
    await state.set_state(DiplomState.waiting_for_topic)
    await message.answer(
        "🎓 **Diplom ishi mavzusini kiriting:**\n\n*(Bot diplom ishi uchun annotatsiya, boblar rejsi va kirish qismini tayyorlab beradi)*",
        reply_markup=ReplyKeyboardRemove(),
        parse_mode="Markdown"
    )


@router.message(DiplomState.waiting_for_topic)
async def process_diplom_topic(message: types.Message, state: FSMContext):
    topic = message.text.strip()
    await state.clear()

    diplom_text = (
        f"🎓 **DIPLOM ISHI LOYIHASI**\n\n"
        f"**Mavzu:** {topic}\n\n"
        f"**Tarkibiy Tuzilmasi:**\n"
        f"• **Annotatsiya:** Mavzuning ilmiy yangiligi va amaliy ahamiyati.\n"
        f"• **1-bob:** Nazariy-metodologik asoslar.\n"
        f"• **2-bob:** Tahliliy qism va mavjud holat.\n"
        f"• **3-bob:** Taklif va tavsiyalar, samaradorlik.\n\n"
        f"📌 *Ushbu loyiha bo'yicha to'liq ma'lumot va sahifalar tayyorlash uchun administratorga murojaat qiling.*"
    )

    await message.answer(diplom_text, reply_markup=get_main_menu(), parse_mode="Markdown")


# --- 📝 Mustaqil ish ---
@router.message(F.text == "📝 Mustaqil ish")
async def cmd_mustaqil(message: types.Message, bot: Bot, state: FSMContext):
    if not await check_subscription(bot, message.from_user.id):
        await message.answer("⚠️ Avval kanallarga obuna bo'ling!", reply_markup=get_sub_keyboard())
        return
    await state.set_state(MustaqilIshState.waiting_for_topic)
    await message.answer(
        "📝 **Mustaqil ish mavzusini kiriting:**",
        reply_markup=ReplyKeyboardRemove(),
        parse_mode="Markdown"
    )


@router.message(MustaqilIshState.waiting_for_topic)
async def process_mustaqil_topic(message: types.Message, state: FSMContext):
    topic = message.text.strip()
    await state.clear()

    mustaqil_text = (
        f"📝 **MUSTAQIL ISH**\n\n"
        f"**Mavzu:** {topic}\n\n"
        f"**1. Kirish:** Mavzuning dolzarblik darajasi.\n"
        f"**2. Asosiy qism:** {topic} bo'yicha tahliliy yondashuvlar va sharh.\n"
        f"**3. Xulosa:** Olingan natijalar va shaxsiy yondashuv.\n"
        f"**4. Foydalanilgan adabiyotlar ro'yxati.**"
    )

    await message.answer(mustaqil_text, reply_markup=get_main_menu(), parse_mode="Markdown")


# --- 👤 Profil ---
@router.message(F.text == "👤 Profil")
async def cmd_profile(message: types.Message, bot: Bot):
    if not await check_subscription(bot, message.from_user.id):
        await message.answer("⚠️ Avval kanallarga obuna bo'ling!", reply_markup=get_sub_keyboard())
        return

    user = message.from_user
    profile_text = (
        f"👤 **Foydalanuvchi Profili:**\n\n"
        f"🆔 **ID:** `{user.id}`\n"
        f"👤 **Ism:** {user.full_name}\n"
        f"🔗 **Username:** @{user.username if user.username else 'Kiritilmagan'}\n"
        f"📢 **Obuna holati:** Tasdiqlangan ✅"
    )
    await message.answer(profile_text, reply_markup=get_main_menu(), parse_mode="Markdown")


# -------------------------------------------------------------
# MAIN RUNNER
# -------------------------------------------------------------
async def main():
    bot = Bot(token=BOT_TOKEN)
    dp = Dispatcher(storage=MemoryStorage())
    dp.include_router(router)

    # Botni ishga tushirish
    await bot.delete_webhook(drop_pending_updates=True)
    await dp.start_polling(bot)


if __name__ == "__main__":
    asyncio.run(main())
