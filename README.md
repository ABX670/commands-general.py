from database import get_user
from config import CURRENCY
import random
from datetime import datetime, timedelta

STORE_ITEMS = {
    "تغيير اللقب": {"price": 1200, "emoji": "🏷️", "desc": "تغيير لقبك او غيرك"},
    "بطاقة الحظ": {"price": 200, "emoji": "🍀", "desc": "تعطي فلوس حسب حظك"},
    "اختبار الإشراف": {"price": 5000, "emoji": "👑", "desc": "عمل اختبار لك لتصبح مشرف"},
    "حماية": {"price": 2500, "emoji": "🛡️", "desc": "حماية من الانذارات والطرد لمدة يوم"},
    "VIP عادية": {"price": 4500, "emoji": "👑", "desc": "رتبة تمنحك تقدير عالي"},
    "VIP سيلفر": {"price": 7500, "emoji": "🥈", "desc": "تقدير اعلى وامكانيات اكثر"},
    "VIP ذهبية": {"price": 12000, "emoji": "🥇", "desc": "تقدير عالي جدا"},
    "VIP رينبو": {"price": 25000, "emoji": "🌈", "desc": "ثاني افضل رتبة"},
    "PREMIUM اسطورية": {"price": 500000, "emoji": "⭐", "desc": "افضل رتبة تجعلك مشرف"},
    "صندوق غنائم الحظ": {"price": 1000, "emoji": "🎁", "desc": "حظه ضعف بطاقة الحظ ويعطي 3 جوايز"},
    "مضاعف XP يوم": {"price": 800, "emoji": "⚡", "desc": "يضاعف XP لمدة 24 ساعة"},
    "كتم حماية": {"price": 1800, "emoji": "🔇", "desc": "حماية من الكتم لمدة يوم"},
    "هدية عشوائية": {"price": 600, "emoji": "🎀", "desc": "ترسل هدية عشوائية لشخص"},
    "تغيير لون الاسم": {"price": 1500, "emoji": "🎨", "desc": "تغيير لون اسمك في التوب"},
    "سرقة محمية": {"price": 3000, "emoji": "💸", "desc": "سرقة 10% من فلوس شخص 50% نجاح"},
    "جواز سفر مزور": {"price": 7000, "emoji": "🛂", "desc": "فك حظر الألعاب"},
}

STORE = {k: v["price"] for k, v in STORE_ITEMS.items()}

def build_store_message():
    return """*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
*┆⌝ Kingdom Meteors Store ⌞┆*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
> ┆*━━━━━━━━━━━━━━━*
> ┆_* تغيير اللقب*_ - *1200$* ↼تغيير لقبك او غيرك
> ┆_* بطاقة الحظ*_ - *200$* ↼تعطي فلوس حسب حظك
> ┆_* اختبار الإشراف*_ - *5000$* ↼عمل اختبار لك لتصبح مشرف
> ┆_* حماية*_ - *2500$* ↼حماية من الانذارات والطرد لمدة يوم
> ┆_* VIP عادية*_ - *4500$* ↼رتبة تمنحك تقدير عالي و اشياء اخرى
> ┆_* VIP سيلفر*_ - *7500$* ↼تمنحك تقدير اعلى وامكانيات اكثر
> ┆_* VIP ذهبية*_ - *12000$* ↼تقدير عالي جدا وامكانيات كثيرة جدا
> ┆_* VIP رينبو*_ - *25000$* ↼ثاني افضل رتبة فالمتجر امكانياتها اسطوريه
> ┆_* PREMIUM اسطورية*_ - *500000$* ↼رتبة اسطورية افضل رتبة ممكن تجعلك مشرف
> ┆_* صندوق غنائم الحظ*_ - *1000$* ↼حظه ضعف بطاقة الحظ ويعطي ثلاث جوايز
> ┆_* مضاعف XP يوم*_ - *800$* ↼يضاعف XP لمدة 24 ساعة
> ┆_* كتم حماية*_ - *1800$* ↼حماية من الكتم لمدة يوم
> ┆_* هدية عشوائية*_ - *600$* ↼ترسل هدية عشوائية لشخص بالجروب
> ┆_* تغيير لون الاسم*_ - *1500$* ↼تغيير لون اسمك في التوب
> ┆_* سرقة محمية*_ - *3000$* ↼محاولة سرقة 10% من فلوس شخص (نسبة نجاح 50%)
> ┆_* جواز سفر مزور*_ - *7000$* ↼فك حظر الألعاب عن نفسك مرة واحدة
> ┆*━━━━━━━━━━━━━━━*
> ┆ _للشراء:_ *اشتري + اسم الصنف*
> ┆ _مثال:_ *اشتري بطاقة الحظ*
> ┆*━━━━━━━━━━━━━━━*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
*┇☄️ Kingdom Meteors Store ☄️┇*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*"""

def handle_general(db, sender, text, mentions=[]):
    t = text.strip()
    u = get_user(db, sender)
    if "bag" not in u: u["bag"] = []
    if "balance" not in u: u["balance"] = 100

    if t == "متجر":
        return build_store_message()

    if t.startswith("اشتري"):
        item_name = t.replace("اشتري","").strip()
        # مطابقة ذكية
        if item_name not in STORE_ITEMS:
            low = item_name.lower()
            for key in STORE_ITEMS.keys():
                if key.lower() in low or low in key.lower():
                    item_name = key; break
                if "رينبو" in low and "رينبو" in key.lower(): item_name = key; break
                if "ذهبي" in low and "ذهبية" in key: item_name = key; break
                if "سيلفر" in low and "سيلفر" in key.lower(): item_name = key; break
                if "premium" in low and "premium" in key.lower(): item_name = key; break
                if "غنائم" in low and "غنائم" in key: item_name = key; break
                if "جواز" in low and "جواز" in key: item_name = key; break
                if "سرقة" in low and "سرقة" in key: item_name = key; break
                if "لون" in low and "لون" in key: item_name = key; break
                if "لقب" in low and "لقب" in key: item_name = key; break
                if "حظ" in low and key == "بطاقة الحظ": item_name = key; break
                if "كتم" in low and "كتم" in key: item_name = key; break
                if "هدية" in low and "هدية" in key: item_name = key; break
                if "حماية" == low and key == "حماية": item_name = key; break

        if item_name not in STORE_ITEMS:
            return f"❌ الصنف '{item_name}' مش موجود\nاكتب *متجر*"

        price = STORE_ITEMS[item_name]["price"]
        if u["balance"] < price:
            return f"❌ فلوسك لا تكفي، تحتاج {price}$ ومعك {u['balance']}$"

        u["balance"] -= price
        emoji = STORE_ITEMS[item_name]["emoji"]

        if item_name == "تغيير اللقب":
            u["pending_title_change"] = True
            return f"{emoji} ✅ اشتريت تغيير اللقب\nاكتب: *لقبي [اللقب الجديد]*\nمثال: لقبي ملك النيازك"

        elif item_name == "بطاقة الحظ":
            win = random.randint(50, 800)
            if random.random() < 0.25: win = -random.randint(20, 120)
            u["balance"] += win
            u["xp"] = u.get("xp",0)+10
            if win >=0:
                return f"🍀 حظك حلو! كسبت {win}$\n💰 رصيدك: {u['balance']}$"
            else:
                return f"🍀 حظك سيء، خسرت {abs(win)}$\n💰 رصيدك: {u['balance']}$"

        elif item_name == "صندوق غنائم الحظ":
            wins = [random.randint(200,1500) for _ in range(3)]
            total = sum(wins)
            u["balance"] += total
            u["xp"] = u.get("xp",0)+25
            return f"🎁 *صندوق الغنائم انفجر!*\n{wins[0]}$ + {wins[1]}$ + {wins[2]}$ = *{total}$*\n💰 رصيدك: {u['balance']}$"

        elif item_name == "اختبار الإشراف":
            u["bag"].append(item_name)
            u["test_bought"] = True
            return "👑 اشتريت اختبار الإشراف!\nاكتب *اختباري* عشان تبدأ\nلازم تجيب 4/5 صح"

        elif item_name == "حماية":
            u["protection_until"] = (datetime.now() + timedelta(days=1)).isoformat()
            u["bag"].append("حماية")
            return "🛡️ حماية مفعلة 24 ساعة ضد الطرد والانذارات"

        elif "VIP" in item_name or "PREMIUM" in item_name:
            u["rank"] = item_name
            u["bag"].append(item_name)
            if "PREMIUM" in item_name:
                return "⭐🔥 مبروك PREMIUM اسطورية! اقوى رتبة! كلم المالك تبقى مشرف"
            return f"{emoji} مبروك رتبة {item_name} تم تفعيلها!"

        elif item_name == "مضاعف XP يوم":
            u["xp_boost_until"] = (datetime.now() + timedelta(days=1)).isoformat()
            u["xp_boost"] = 2
            return "⚡ مضاعف XP x2 مفعل لمدة يوم كامل!"

        elif item_name == "كتم حماية":
            u["mute_protection_until"] = (datetime.now() + timedelta(days=1)).isoformat()
            return "🔇 حماية من الكتم مفعلة 24 ساعة"

        elif item_name == "هدية عشوائية":
            if not mentions:
                # يشتريها ويهديها
                u["bag"].append("هدية عشوائية")
                return "🎀 اشتريت هدية عشوائية! اكتب: اهداء @منشن"
            else:
                gift = random.randint(100,500)
                target = get_user(db, mentions[0])
                target["balance"] += gift
                return f"🎀 أهديت {gift}$ لـ @{mentions[0]}!"

        elif item_name == "تغيير لون الاسم":
            u["pending_color_change"] = True
            return "🎨 اكتب: لوني [احمر/ازرق/اخضر/ذهبي/رينبو]\nمثال: لوني ذهبي"

        elif item_name == "سرقة محمية":
            u["bag"].append("سرقة محمية")
            return "💸 اشتريت سرقة محمية!\nاكتب: اسرق @منشن\nنسبة نجاح 50%"

        elif item_name == "جواز سفر مزور":
            if u.get("banned_game"):
                u["banned_game"] = False
                return "🛂 فكيت حظر الألعاب عنك! تقدر تلعب دلوقتي"
            else:
                u["bag"].append(item_name)
                return "🛂 اشتريت جواز سفر، هيتحفظ للطوارئ"

    # باقي أوامر المتجر
    if t == "الشنطة":
        bag = u.get("bag", [])
        if not bag: return "🎒 الشنطة فاضية"
        from collections import Counter
        c = Counter(bag)
        msg = "🎒 *شنطتك:*\n"
        for k,v in c.items():
            msg+=f"• {k} x{v}\n"
        msg+=f"\n💰 {u['balance']}$"
        return msg

    if t == "ملف":
        rank = u.get("rank","عضو عادي")
        title = u.get("title","بدون")
        prot = "🟢" if u.get("protection_until") else "🔴"
        mprot = "🟢" if u.get("mute_protection_until") else "🔴"
        return f"📄 ملفك\n💰 {u['balance']}$\n⭐ لفل {u.get('level',1)} XP {u.get('xp',0)}\n👑 {rank}\n🏷️ {title}\n🛡️ طرد:{prot} 🔇 كتم:{mprot}\n🎒 {len(u.get('bag',[]))} عناصر"

    if t.startswith("لقبي"):
        if not u.get("pending_title_change") and "تغيير اللقب" not in u.get("bag",[]):
            return "❌ اشتري تغيير اللقب أولا"
        new_title = t.replace("لقبي","").strip()
        if len(new_title)<2 or len(new_title)>25: return "❌ اللقب من 2 لـ 25 حرف"
        u["title"] = new_title
        u["pending_title_change"] = False
        if "تغيير اللقب" in u["bag"]: u["bag"].remove("تغيير اللقب")
        return f"✅ لقبك الجديد: {new_title}"

    if t.startswith("لوني"):
        if not u.get("pending_color_change"): return "❌ اشتري تغيير لون الاسم أولا"
        color = t.replace("لوني","").strip()
        allowed = ["احمر","ازرق","اخضر","ذهبي","رينبو","بنفسجي","اسود"]
        if color.lower() not in allowed: return f"الالوان: {', '.join(allowed)}"
        u["name_color"] = color
        u["pending_color_change"] = False
        return f"🎨 لونك الجديد: {color}"

    if t.startswith("اسرق") and mentions:
        if "سرقة محمية" not in u.get("bag",[]):
            return "❌ اشتري سرقة محمية أولا (3000$)"
        target = get_user(db, mentions[0])
        if target["balance"] < 20: return f"@{mentions[0]} فقير"
        if random.choice([True,False]):
            steal = int(target["balance"]*0.1)
            steal = max(10, min(steal,1000))
            target["balance"]-=steal; u["balance"]+=steal
            u["bag"].remove("سرقة محمية")
            return f"💸 سرقت {steal}$ من @{mentions[0]}!\n💰 رصيدك: {u['balance']}$"
        else:
            u["bag"].remove("سرقة محمية")
            return f"😂 فشلت سرقة @{mentions[0]} واتفضحت!"

    if t.startswith("اهداء") and mentions:
        target = get_user(db, mentions[0])
        if "هدية عشوائية" in u["bag"]:
            gift = random.randint(100,500)
            target["balance"]+=gift
            u["bag"].remove("هدية عشوائية")
            return f"🎀 أهديت {gift}$ لـ @{mentions[0]}"
        return "❌ معكش هدية عشوائية"

    if t == "اختباري":
        if not u.get("test_bought") and "اختبار الإشراف" not in u.get("bag",[]):
            return "❌ اشتري اختبار الإشراف أولا"
        return """📜 اختبار الإشراف
1- عاصمة مصر؟ جواب1 القاهرة
2- 2+2؟ جواب2 4
3- بطل ون بيس؟ جواب3 لوفي
4- امر المتجر؟ جواب4 متجر
5- سعر VIP رينبو؟ جواب5 25000"""

    if t.startswith("منح") and mentions:
        try:
            amount = int(t.split()[1])
            if u["balance"] < amount: return "❌ فلوسك لا تكفي"
            u["balance"]-=amount
            get_user(db, mentions[0])["balance"]+=amount
            return f"🎁 منحت {amount}$ لـ @{mentions[0]}"
        except:
            return "منح 100 @منشن"

    if t == "توب":
        users = sorted(db["users"].items(), key=lambda x: x[1].get("balance",0), reverse=True)[:10]
        if not users: return "لا يوجد لاعبين"
        msg="🏆 أغنى 10:\n"
        for i,(num,d) in enumerate(users):
            msg+=f"{i+1}. @{num} {d['balance']}$ {d.get('rank','')} {d.get('title','')}\n"
        return msg

    if t.startswith("رهان"):
        try:
            amount=int(t.split()[1])
            if u["balance"]<amount: return "❌ فلوسك لا تكفي"
            if random.choice([True,False]):
                u["balance"]+=amount; return f"🎉 كسبت {amount}$"
            else:
                u["balance"]-=amount; return f"😢 خسرت {amount}$"
        except:
            return "رهان 50"

    return None
    
