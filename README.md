from database import get_user
from config import CURRENCY
import random
from datetime import datetime, timedelta

# ============ متجر Kingdom Meteors - النسخة المحسنة النهائية ============

STORE_ITEMS = {
    "تغيير اللقب": {"price": 1200, "emoji": "🏷️", "desc": "تغيير لقبك او غيرك"},
    "بطاقة الحظ": {"price": 200, "emoji": "🍀", "desc": "تعطي فلوس حسب حظك"},
    "اختبار الإشراف": {"price": 5000, "emoji": "👑", "desc": "عمل اختبار لك لتصبح مشرف"},
    "حماية": {"price": 2500, "emoji": "🛡️", "desc": "حماية من الانذارات والطرد لمدة يوم"},
    "VIP عادية": {"price": 4500, "emoji": "👑", "desc": "رتبة تمنحك تقدير عالي و اشياء اخرى"},
    "VIP سيلفر": {"price": 7500, "emoji": "🥈", "desc": "تمنحك تقدير اعلى وامكانيات اكثر"},
    "VIP ذهبية": {"price": 12000, "emoji": "🥇", "desc": "تقدير عالي جدا وامكانيات كثيرة جدا"},
    "VIP رينبو": {"price": 25000, "emoji": "🌈", "desc": "ثاني افضل رتبة فالمتجر امكانياتها اسطوريه"},
    "PREMIUM اسطورية": {"price": 500000, "emoji": "⭐", "desc": "افضل رتبة ممكن تجعلك مشرف"},
    "صندوق غنائم الحظ": {"price": 1000, "emoji": "🎁", "desc": "حظه ضعف بطاقة الحظ ويعطي ثلاث جوايز"},
    "مضاعف XP يوم": {"price": 800, "emoji": "⚡", "desc": "يضاعف XP لمدة 24 ساعة"},
    "كتم حماية": {"price": 1800, "emoji": "🔇", "desc": "حماية من الكتم لمدة يوم"},
    "هدية عشوائية": {"price": 600, "emoji": "🎀", "desc": "ترسل هدية عشوائية لشخص بالجروب"},
    "تغيير لون الاسم": {"price": 1500, "emoji": "🎨", "desc": "تغيير لون اسمك في التوب"},
    "سرقة محمية": {"price": 3000, "emoji": "💸", "desc": "محاولة سرقة 10% من فلوس شخص (50% نجاح)"},
    "جواز سفر مزور": {"price": 7000, "emoji": "🛂", "desc": "فك حظر الألعاب عن نفسك مرة واحدة"},
}

STORE = {k: v["price"] for k, v in STORE_ITEMS.items()}

def build_store_message():
    # القايمة المحسنة - نفس طلبك بس مرتبة احسن
    return """*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
*┆⌝ Kingdom Meteors Store ⌞┆*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
> ┆*━━━━━━━━━━━━━━━*
> ┆ *`[المتجر العام]`*
> ┆*━━━━━━━━━━━━━━━*
> ┆ 🏷️ _*تغيير اللقب*_ - *1200$*
> ┆ ↼ تغيير لقبك او غيرك
> ┆
> ┆ 🍀 _*بطاقة الحظ*_ - *200$*
> ┆ ↼ تعطي فلوس حسب حظك من 50$ ل 800$
> ┆
> ┆ 👑 _*اختبار الإشراف*_ - *5000$*
> ┆ ↼ عمل اختبار لك لتصبح مشرف
> ┆
> ┆ 🛡️ _*حماية*_ - *2500$*
> ┆ ↼ حماية من الانذارات والطرد لمدة يوم كامل
> ┆*━━━━━━━━━━━━━━━*
> ┆ *`[رتب الـ VIP]`*
> ┆*━━━━━━━━━━━━━━━*
> ┆ 👑 _*VIP عادية*_ - *4500$*
> ┆ 🥈 _*VIP سيلفر*_ - *7500$*
> ┆ 🥇 _*VIP ذهبية*_ - *12000$*
> ┆ 🌈 _*VIP رينبو*_ - *25000$*
> ┆ ⭐ _*PREMIUM اسطورية*_ - *500000$*
> ┆*━━━━━━━━━━━━━━━*
> ┆ *`[صناديق و حظ]`*
> ┆*━━━━━━━━━━━━━━━*
> ┆ 🎁 _*صندوق غنائم الحظ*_ - *1000$*
> ┆ ↼ حظه ضعف بطاقة الحظ ويعطي ثلاث جوايز
> ┆
> ┆ ⚡ _*مضاعف XP يوم*_ - *800$*
> ┆ ↼ يضاعف XP لمدة 24 ساعة
> ┆
> ┆ 🔇 _*كتم حماية*_ - *1800$*
> ┆ ↼ حماية من الكتم لمدة يوم
> ┆
> ┆ 🎀 _*هدية عشوائية*_ - *600$*
> ┆ ↼ ترسل هدية عشوائية لشخص بالجروب
> ┆
> ┆ 🎨 _*تغيير لون الاسم*_ - *1500$*
> ┆ ↼ تغيير لون اسمك في التوب والملف
> ┆
> ┆ 💸 _*سرقة محمية*_ - *3000$*
> ┆ ↼ محاولة سرقة 10% من فلوس شخص (نسبة نجاح 50%)
> ┆
> ┆ 🛂 _*جواز سفر مزور*_ - *7000$*
> ┆ ↼ فك حظر الألعاب عن نفسك مرة واحدة
> ┆*━━━━━━━━━━━━━━━*
> ┆ _للشراء اكتب:_ *اشتري + اسم الصنف*
> ┆ _مثال:_ *اشتري بطاقة الحظ*
> ┆*━━━━━━━━━━━━━━━*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*
*┇☄️ Kingdom Meteors Store ☄️┇*
*╔⟐━─⧉━⌬〔☄️〕⌬━⧉─━⟐╗*"""

def handle_general(db, sender, text, mentions=[]):
    t = text.strip()
    u = get_user(db, sender)

    # تأكد من وجود الحقول
    if "bag" not in u: u["bag"] = []
    if "balance" not in u: u["balance"] = 100

    if t == "متجر":
        return build_store_message()

    if t.startswith("اشتري"):
        item_name = t.replace("اشتري", "").strip()
        
        # مطابقة ذكية
        if item_name not in STORE_ITEMS:
            # ازالة ايموجي ورموز
            low = item_name.lower()
            for key in STORE_ITEMS.keys():
                k_low = key.lower()
                if k_low in low or low in k_low:
                    item_name = key
                    break
                # حالات خاصة
                if "رينبو" in low and "رينبو" in k_low: item_name = key; break
                if "ذهبي" in low and "ذهبية" in k_low: item_name = key; break
                if "سيلفر" in low and "سيلفر" in k_low: item_name = key; break
                if "premium" in low and "premium" in k_low: item_name = key; break
                if "غنائم" in low and "غنائم" in k_low: item_name = key; break
                if "حماية" in low and key == "حماية": item_name = key; break
                if "كتم" in low and "كتم" in k_low: item_name = key; break
                if "جواز" in low and "جواز" in k_low: item_name = key; break
                if "سرقة" in low and "سرقة" in k_low: item_name = key; break
                if "لون" in low and "لون" in k_low: item_name = key; break
                if "لقب" in low and "تغيير اللقب" in k_low: item_name = key; break

        if item_name not in STORE_ITEMS:
            return f"❌ الصنف '{item_name}' مش موجود\nاكتب *متجر* عشان تشوف القايمة"

        price = STORE_ITEMS[item_name]["price"]
        if u["balance"] < price:
            return f"❌ فلوسك لا تكفي\nتحتاج {price}$ ومعك {u['balance']}$\nاكسب من الألعاب: تفكيك / اعلام / اسرع"

        u["balance"] -= price
        emoji = STORE_ITEMS[item_name]["emoji"]

        # ====== التابات المبرمجة لكل صنف ======
        
        if item_name == "تغيير اللقب":
            u["pending_title_change"] = True
            return f"{emoji} ✅ اشتريت *تغيير اللقب* بـ {price}$\nالآن اكتب:\n*لقبي [اللقب الجديد]*\nمثال: لقبي ملك الظلام 👑"

        elif item_name == "بطاقة الحظ":
            win = random.randint(50, 800)
            # 20% تخسر
            if random.random() < 0.2:
                win = -random.randint(20, 100)
            u["balance"] += win
            u["xp"] = u.get("xp", 0) + 10
            if win >= 0:
                return f"🍀 حظك حلو! كسبت *{win}$* من بطاقة الحظ!\n💰 رصيدك الآن: {u['balance']}$"
            else:
                return f"🍀 اوبس حظك وحش، خسرت {abs(win)}$\n💰 رصيدك: {u['balance']}$"

        elif item_name == "اختبار الإشراف":
            u["bag"].append(item_name)
            u["test_bought"] = True
            return f"👑 اشتريت اختبار الإشراف!\n📜 اكتب *اختباري* لبدء الاختبار\n(5 أسئلة - لازم تجيب 4 صح)\nلو نجحت هتبقى مشرف!"

        elif item_name == "حماية":
            u["protection_until"] = (datetime.now() + timedelta(days=1)).isoformat()
            u["bag"].append("حماية")
            return f"🛡️ تم تفعيل الحماية لمدة 24 ساعة!\nمحدش يقدر يطردك او ينذرك\n⏰ تنتهي: بعد يوم"

        elif "VIP" in item_name or "PREMIUM" in item_name:
            u["rank"] = item_name
            u["bag"].append(item_name)
            if "PREMIUM" in item_name:
                return f"⭐🔥 مبرووووك! حصلت على *{item_name}*\nدي أقوى رتبة في السيرفر!\n👑 مميزات: فلوس كتير + احترام + هتبقى مشرف قريب\nتواصل مع المالك!"
            elif "رينبو" in item_name:
                return f"🌈 مبروك رتبة *VIP رينبو* !\nرتبة اسطورية وامكانياتها خارقة!"
            else:
                return f"{emoji} تم تفعيل رتبة *{item_name}* بنجاح!\nهتظهر في *توب* و *ملف*"

        elif item_name == "صندوق غنائم الحظ":
            wins = [random.randint(200, 1500) for _ in range(3)]
            total = sum(wins)
            u["balance"] += total
            u["bag"].append(item_name)
            u["xp"] = u.get("xp",0) + 20
            return f"🎁🎁 *صندوق الغنائم انفجر!*\n💰 {wins[0]}$ + {wins[1]}$ + {wins[2]}$\n= *{total}$* كسبتهم!\n💰 رصيدك: {u['balance']}$"

        elif item_name == "مضاعف XP يوم":
            u["xp_boost_until"] = (datetime.now() + timedelta(days=1)).isoformat()
            u["xp_boost"] = 2
            return f"⚡ تم تفعيل مضاعف XP x2 لمدة يوم!\nكل لعبة هتاخد ضعف ال XP"

        elif item_name == "كتم حماية":
            u["mute_protection_until"] = (datetime.now() + timedelta(days=1)).isoformat()
            return f"🔇 تم تفعيل حماية الكتم لمدة 24 ساعة\nمحدش يقدر يكتمك"

        elif item_name == "هدية عشوائية":
            # يختار هدية عشوائية
            gifts = ["سيف", "درع", "100$", "200$", "50 XP"]
            gift = random.choice(gifts)
            return f"🎀 اشتريت هدية عشوائية ({gift})\nاكتب: *اهداء @منشن* عشان تهديها\nمثال: اهداء هدية عشوائية @احمد"

        elif item_name == "تغيير لون الاسم":
            u["pending_color_change"] = True
            return f"🎨 اشتريت تغيير لون الاسم!\nاكتب: *لوني [اللون]*\nالالوان: احمر / ازرق / اخضر / ذهبي / رينبو\nمثال: لوني ذهبي"

        elif item_name == "سرقة محمية":
            u["bag"].append("سرقة محمية")
            return f"💸 اشتريت سرقة محمية!\nاكتب: *اسرق @منشن*\nنسبة النجاح 50% وتسرق 10% من فلوسه\nلو فشلت هتتفضح!"

        elif item_name == "جواز سفر مزور":
            if u.get("banned_game"):
                u["banned_game"] = False
                return f"🛂 تم فك حظر الألعاب عنك بنجاح!\nتقدر تلعب دلوقتي: تفكيك / اعلام / اسرع"
            else:
                u["bag"].append(item_name)
                return f"🛂 اشتريت جواز سفر مزور\nانت مش محظور حاليا، هيتحفظ للطوارئ"

        else:
            u["bag"].append(item_name)
            return f"✅ اشتريت {item_name} بـ {price}$"

    # ============ أوامر اضافية للمتجر ============
    
    if t == "الشنطة":
        bag = u.get("bag", [])
        if not bag:
            return "🎒 الشنطة فارغة\nاكتب *متجر* عشان تشتري"
        counts = {}
        for item in bag:
            counts[item] = counts.get(item, 0) + 1
        msg = "🎒 *شنطتك:*\n"
        for item, count in counts.items():
            msg += f"• {item} x{count}\n"
        msg += f"\n💰 رصيدك: {u['balance']}$"
        return msg

    if t == "ملف":
        rank = u.get("rank", "عضو عادي")
        title = u.get("title", "بدون لقب")
        color = u.get("name_color", "عادي")
        prot = "🟢 مفعل" if u.get("protection_until") else "🔴 غير مفعل"
        mute_prot = "🟢" if u.get("mute_protection_until") else "🔴"
        return f"""📄 *ملفك في Kingdom Meteors*
*╔━━━━━━━━━━━━━━━╗*
💰 فلوسك: {u['balance']}$
⭐ لفل: {u.get('level',1)} | XP: {u.get('xp',0)}
🏆 انتصارات: {u.get('wins',0)}
👑 رتبتك: {rank}
🏷️ لقبك: {title}
🎨 لونك: {color}
🛡️ حماية طرد: {prot}
🔇 حماية كتم: {mute_prot}
🎒 الشنطة: {len(u.get('bag',[]))} عناصر
🧬 جنسك: {u.get('gender','غير محدد')}
*╚━━━━━━━━━━━━━━━╝*"""

    if t.startswith("لقبي"):
        if not u.get("pending_title_change") and "تغيير اللقب" not in u.get("bag",[]):
            return "❌ لازم تشتري تغيير اللقب أولا\nاكتب: اشتري تغيير اللقب (1200$)"
        new_title = t.replace("لقبي", "").strip()
        if len(new_title) < 2:
            return "❌ اللقب قصير، اكتب لقب اطول\nمثال: لقبي ملك الظلام"
        if len(new_title) > 25:
            return "❌ اللقب طويل جدا (اقصى 25 حرف)"
        u["title"] = new_title
        u["pending_title_change"] = False
        if "تغيير اللقب" in u.get("bag",[]):
            u["bag"].remove("تغيير اللقب")
        return f"✅ تم تغيير لقبك إلى: *{new_title}* 🏷️"

    if t.startswith("لوني"):
        if not u.get("pending_color_change"):
            return "❌ اشتري تغيير لون الاسم أولا (1500$)"
        color = t.replace("لوني", "").strip().lower()
        allowed = ["احمر", "ازرق", "اخضر", "ذهبي", "رينبو", "بنفسجي", "اسود"]
        if color not in allowed:
            return f"❌ اللون مش موجود، الالوان المتاحة: {', '.join(allowed)}"
        u["name_color"] = color
        u["pending_color_change"] = False
        return f"🎨 تم تغيير لون اسمك إلى *{color}*"

    if t.startswith("اسرق") and mentions:
        if "سرقة محمية" not in u.get("bag",[]) and not u.get("can_steal"):
            return "❌ لازم تشتري سرقة محمية أولا (3000$)\nاكتب: اشتري سرقة محمية"
        target_id = mentions[0]
        target = get_user(db, target_id)
        # حماية الدرع
        if target.get("shield_until"):
            try:
                until = datetime.fromisoformat(target["shield_until"])
                if datetime.now() < until:
                    return f"🛡️ @{target_id} عنده درع أسطوري! مقدرتش تسرقه"
            except:
                pass
        
        if random.choice([True, False]):  # 50% نجاح
            steal_amount = int(target["balance"] * 0.1)
            steal_amount = max(10, min(steal_amount, 1000))  # بين 10 و 1000
            if target["balance"] < steal_amount:
                return f"😢 @{target_id} فقير معهوش فلوس تسرقها"
            target["balance"] -= steal_amount
            u["balance"] += steal_amount
            if "سرقة محمية" in u["bag"]:
                u["bag"].remove("سرقة محمية")
            return f"💸 نجحت! سرقت *{steal_amount}$* من @{target_id} 🤑\n💰 رصيدك: {u['balance']}$"
        else:
            if "سرقة محمية" in u["bag"]:
                u["bag"].remove("سرقة محمية")
            return f"😂 فشلت سرقة @{target_id} واتفضحت!\nالمرة الجاية حاول تبقى محترف"

    if t.startswith("اهداء") and mentions:
        target_id = mentions[0]
        target = get_user(db, target_id)
        if not u.get("bag"):
            return "❌ شنطتك فاضية"
        # لو كاتب اهداء هدية عشوائية
        if "هدية عشوائية" in t or "هدية" in u["bag"]:
            gift_value = random.randint(50, 300)
            target["balance"] += gift_value
            if "هدية عشوائية" in u["bag"]:
                u["bag"].remove("هدية عشوائية")
            return f"🎀 أهديت @{target_id} هدية بقيمة {gift_value}$! 🎁"
        else:
            # اهداء عادي
            parts = t.split()
            if len(parts) >= 2:
                item = parts[1]
                if item in u["bag"]:
                    u["bag"].remove(item)
                    target["bag"].append(item)
                    return f"🎁 أهديت {item} إلى @{target_id}"
            return "❌ اكتب: اهداء @منشن\nمثال: اهداء هدية عشوائية @احمد"

    if t == "اختباري":
        if not u.get("test_bought") and "اختبار الإشراف" not in u.get("bag",[]):
            return "❌ اشتري اختبار الإشراف أولا (5000$)"
        return """📜 *اختبار الإشراف - Kingdom Meteors*
*━━━━━━━━━━━━━━━*
1️⃣ ما هي عاصمة مصر؟
اكتب: جواب1 القاهرة

2️⃣ كم 2+2؟
اكتب: جواب2 4

3️⃣ من هو بطل ون بيس؟
اكتب: جواب3 لوفي

4️⃣ ما هو امر المتجر؟
اكتب: جواب4 متجر

5️⃣ كم سعر VIP رينبو؟
اكتب: جواب5 25000

_لازم تجيب 4/5 صح_
*━━━━━━━━━━━━━━━*"""

    if t.startswith("جواب"):
        # نظام بسيط للاختبار
        if "اختبار الإشراف" not in u.get("bag",[]):
            return "❌ اشتري الاختبار أولا"
        # هنا ممكن تطور نظام الدرجات
        return f"✅ تم تسجيل {t}\nكمل باقي الأسئلة"

    # أوامر قديمة
    if t.startswith("منح") and mentions:
        try:
            parts = t.split()
            amount = int(parts[1])
            if u["balance"] < amount:
                return "❌ فلوسك لا تكفي"
            if amount <= 0:
                return "❌ المبلغ لازم يكون موجب"
            u["balance"] -= amount
            target_id = mentions[0]
            get_user(db, target_id)["balance"] += amount
            return f"🎁 تم منح {amount}$ لـ @{target_id}"
        except Exception as e:
            return "❌ اكتب: منح 100 @منشن"

    if t.startswith("AB"):
        try:
            amount = int(t.split()[1])
            u["balance"] += amount
            return f"💰 أضفت لنفسك {amount}$ (للمطورين فقط)"
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
    t = text.strip()
    u = get_user(db, sender)
    if t == "توب":
        users = sorted(db["users"].items(), key=lambda x: x[1].get("balance",0), reverse=True)[:10]
        if not users:
            return "لا يوجد لاعبين بعد"
        msg = "🏆 *أغنى 10 في المملكة:*\n*━━━━━━━━━━━━━━━*\n"
        for i,(num,d) in enumerate(users):
            rank = d.get('rank','عضو')
            title = d.get('title','')
            color = d.get('name_color','')
            msg += f"{i+1}. @{num} - {d['balance']}$ | {rank} {title} {f'[{color}]' if color else ''}\n"
        return msg
    if t.startswith("رهان"):
        parts = t.split()
        if len(parts)>=2:
            try: amount=int(parts[1])
            except: return "اكتب: رهان 50"
            if u["balance"]<amount: return "❌ فلوسك لا تكفي"
            if amount < 10: return "❌ اقل رهان 10$"
            if random.choice([True,False]):
                u["balance"]+=amount; u["xp"]=u.get("xp",0)+10
                return f"🎉 كسبت الرهان! +{amount}$\n💰 رصيدك: {u['balance']}$"
            else:
                u["balance"]-=amount
                return f"😢 خسرت الرهان! -{amount}$\n💰 رصيدك: {u['balance']}$"
        return "اكتب: رهان 50"
    return None
