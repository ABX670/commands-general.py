from database import get_user
from config import CURRENCY
import random, time
from datetime import datetime, timedelta

STORE_ITEMS = {
    "تغيير اللقب": {"price": 1200, "desc": "تغيير لقبك او غيرك"},
    "بطاقة الحظ": {"price": 200, "desc": "تعطي فلوس حسب حظك"},
    "اختبار الإشراف": {"price": 5000, "desc": "عمل اختبار لك لتصبح مشرف"},
    "حماية": {"price": 2500, "desc": "حماية من الانذارات والطرد لمدة يوم"},
    "VIP عادية": {"price": 4500, "desc": "رتبة تمنحك تقدير عالي و اشياء اخرى"},
    "VIP سيلفر": {"price": 7500, "desc": "تمنحك تقدير اعلى وامكانيات اكثر"},
    "VIP ذهبية": {"price": 12000, "desc": "تقدير عالي جدا وامكانيات كثيرة جدا"},
    "VIP رينبو": {"price": 25000, "desc": "ثاني افضل رتبة فالمتجر امكانياتها اسطوريه"},
    "PREMIUM اسطورية": {"price": 500000, "desc": "رتبة اسطورية افضل رتبة ممكن تجعلك مشرف"},
    "صندوق غنائم الحظ": {"price": 1000, "desc": "حظه ضعف بطاقة الحظ ويعطي ثلاث جوايز"},
    "مضاعف XP يوم": {"price": 800, "desc": "يضاعف XP لمدة 24 ساعة"},
    "كتم حماية": {"price": 1800, "desc": "حماية من الكتم لمدة يوم"},
    "هدية عشوائية": {"price": 600, "desc": "ترسل هدية عشوائية لشخص بالجروب"},
    "تغيير لون الاسم": {"price": 1500, "desc": "تغيير لون اسمك في التوب"},
    "سرقة محمية": {"price": 3000, "desc": "محاولة سرقة 10% من فلوس شخص (نسبة نجاح 50%)"},
    "جواز سفر مزور": {"price": 7000, "desc": "فك حظر الألعاب عن نفسك مرة واحدة"},
}

STORE = {k: v["price"] for k, v in STORE_ITEMS.items()}

def build_store_message():
    header = """*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
*┆⌝Kingdom Meteors Store⌞┆*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
> ┆*━━━━━━━━━━━━━━━*"""
    lines = []
    for name, info in STORE_ITEMS.items():
        price = info["price"]
        desc = info["desc"]
        lines.append(f"> ┆_* {name}*_ *{price}$* *↼{desc}*")
    footer = """> ┆*━━━━━━━━━━━━━━━*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
*┇☄️ Kingdom Meteors Store ☄️┇*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*"""
    return header + "\n" + "\n".join(lines) + "\n" + footer

def handle_general(db, sender, text, mentions=[]):
    t = text.strip()
    u = get_user(db, sender)

    if t == "متجر":
        return build_store_message()

    if t.startswith("اشتري"):
        item_name = t.replace("اشتري", "").strip()
        if item_name not in STORE_ITEMS:
            for k in STORE_ITEMS:
                if k in item_name or item_name in k:
                    item_name = k
                    break
        if item_name not in STORE_ITEMS:
            return f"❌ الصنف '{item_name}' مش موجود، اكتب *متجر*"
        price = STORE_ITEMS[item_name]["price"]
        if u["balance"] < price:
            return f"❌ فلوسك لا تكفي، تحتاج {price}$ ومعك {u['balance']}$"
        u["balance"] -= price

        if item_name == "تغيير اللقب":
            u["pending_title_change"] = True
            return f"✅ اشتريت *{item_name}*، الآن اكتب: لقبي [اللقب الجديد]"

        elif item_name == "بطاقة الحظ":
            win = random.randint(-100, 500)
            u["balance"] += win
            return f"🍀 استخدمت بطاقة الحظ، كسبت {win}$! رصيدك الآن {u['balance']}$"

        elif item_name == "صندوق غنائم الحظ":
            wins = [random.randint(50, 800) for _ in range(3)]
            total = sum(wins)
            u["balance"] += total
            u["bag"].append(item_name)
            return f"🎁 صندوق الغنائم فتح! كسبت: {wins[0]}$ + {wins[1]}$ + {wins[2]}$ = {total}$"

        elif item_name == "حماية":
            u["protection_until"] = (datetime.now() + timedelta(days=1)).isoformat()
            u["bag"].append("حماية")
            return "🌀 تم تفعيل الحماية من الطرد والانذارات لمدة 24 ساعة"

        elif "VIP" in item_name or "PREMIUM" in item_name:
            u["rank"] = item_name
            u["bag"].append(item_name)
            if item_name == "PREMIUM اسطورية":
                return "⭐ مبروك! حصلت على رتبة PREMIUM اسطورية، تواصل مع المالك لتصبح مشرف"
            return f"👑 تم تفعيل رتبة {item_name} بنجاح!"

        elif item_name == "اختبار الإشراف":
            u["bag"].append(item_name)
            return "📜 تم شراء اختبار الإشراف، اكتب *اختباري* لبدء الاختبار (5 أسئلة)"

        else:
            u["bag"].append(item_name)
            return f"✅ اشتريت {item_name} بـ {price}$ وتم إضافته للشنطة"

    if t == "الشنطة":
        bag = u.get("bag", [])
        if not bag:
            return "🎒 الشنطة فارغة، اكتب *متجر*"
        return "🎒 شنطتك:\n- " + "\n- ".join(bag)

    if t == "ملف":
        rank = u.get("rank", "عضو عادي")
        prot = "مفعل 🟢" if u.get("protection_until") else "غير مفعل 🔴"
        return f"📄 *ملفك في Kingdom Meteors*\n💰 فلوسك: {u['balance']}$\n⭐ لفل: {u.get('level',1)} | XP: {u.get('xp',0)}\n🏆 انتصارات: {u.get('wins',0)}\n👑 رتبتك: {rank}\n🌀 حماية: {prot}\n🎒 الشنطة: {len(u.get('bag',[]))} عناصر\n🧬 جنسك: {u.get('gender','غير محدد')}\n📛 لقبك: {u.get('title','بدون')}\n"

    if t.startswith("لقبي"):
        if not u.get("pending_title_change") and "تغيير اللقب" not in u.get("bag",[]):
            return "❌ لازم تشتري تغيير اللقب من المتجر أولا"
        new_title = t.replace("لقبي", "").strip()
        u["title"] = new_title
        u["pending_title_change"] = False
        if "تغيير اللقب" in u.get("bag",[]):
            u["bag"].remove("تغيير اللقب")
        return f"✅ تم تغيير لقبك إلى: {new_title}"

    if t.startswith("منح") and mentions:
        try:
            parts = t.split()
            amount = int(parts[1])
            if u["balance"] < amount:
                return "❌ فلوسك لا تكفي"
            u["balance"] -= amount
            target = mentions[0]
            get_user(db, target)["balance"] += amount
            return f"🎁 تم منح {amount}$ لـ @{target}"
        except:
            return "اكتب: منح 100 @منشن"

    if t.startswith("AB"):
        try:
            amount = int(t.split()[1])
            u["balance"] += amount
            return f"💰 أضفت لنفسك {amount}$"
        except:
            return "اكتب: AB 100"

    if t == "AC":
        u["balance"] = max(0, u["balance"]-50)
        return "➖ خصمت 50 من نفسك"

    r = handle_extra(db, sender, text, mentions)
    if r:
        return r
    return None

def handle_extra(db, sender, text, mentions):
    import random
    t = text.strip()
    u = get_user(db, sender)
    if t == "توب":
        users = sorted(db["users"].items(), key=lambda x: x[1].get("balance",0), reverse=True)[:10]
        lines = [f"{i+1}. @{num} - {d['balance']}$ | {d.get('rank','عضو')}" for i,(num,d) in enumerate(users)]
        return "🏆 *أغنى 10 في المملكة:*\n" + "\n".join(lines) if lines else "لا يوجد لاعبين"
    if t.startswith("رهان"):
        parts = t.split()
        if len(parts)>=2:
            try: amount=int(parts[1])
            except: return "اكتب: رهان 50"
            if u["balance"]<amount: return "❌ فلوسك لا تكفي"
            if random.choice([True,False]):
                u["balance"]+=amount; u["xp"]+=10
                return f"🎉 كسبت الرهان! +{amount}$"
            else:
                u["balance"]-=amount
                return f"😢 خسرت الرهان! -{amount}$"
        return "اكتب: رهان 50"
    if t.startswith("هدية") and mentions:
        parts = t.split()
        if len(parts)>=2:
            item = parts[1] if len(parts)>1 else None
            target = mentions[0]
            if item and item in u.get("bag",[]):
                u["bag"].remove(item)
                get_user(db,target)["bag"].append(item)
                return f"🎁 أهديت {item} إلى @{target}"
            return "❌ الصنف مش في شنطتك"
    return None
    <span class="line"><span style="--shiki-light:#032F62;--shiki-dark:#9ECBFF">*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*</span>
