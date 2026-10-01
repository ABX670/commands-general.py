from database import get_user
from config import CURRENCY
STORE={"سيف":50,"درع":80,"جرعة":20}

def handle_general(db,sender,text,mentions=[]):
    t=text.strip(); u=get_user(db,sender)
    if t=="متجر":
        items="\n".join([f"{k} - {v} {CURRENCY}" for k,v in STORE.items()])
        return f"🛒 متجر الجروب:\n{items}\nللشراء اكتب: اشتري اسم_الصنف"
    if t.startswith("اشتري"):
        item=t.replace("اشتري","").strip()
        if item in STORE and u["balance"]>=STORE[item]:
            u["balance"]-=STORE[item]; u["bag"].append(item)
            return f"✅ اشتريت {item}"
        return "❌ فلوسك لا تكفي أو الصنف غير موجود"
    if t=="ملف":
        return f"📄 ملفك:\n💰 {u['balance']} {CURRENCY}\n⭐ لفل {u['level']} | XP {u['xp']}\n🏆 انتصارات: {u['wins']}\n🎒 الشنطة: {len(u['bag'])} عناصر"
    if t=="الشنطة":
        return "🎒 الشنطة فارغة" if not u["bag"] else "🎒 شنطتك: "+", ".join(u["bag"])
    if t.startswith("منح"):
        parts=t.split(); amount=int(parts[1]); u["balance"]-=amount
        return f"🎁 تم منح {amount} {CURRENCY} (طبق التحويل لمنشن)"
    if t.startswith("AB"):
        amount=int(t.split()[1]); u["balance"]+=amount; return f"💰 أضفت لنفسك {amount}"
    if t=="AC":
        u["balance"]=max(0,u["balance"]-50); return "➖ خصمت 50 من نفسك"
    r=handle_extra(db,sender,text,mentions)
    if r: return r
    return None

def handle_extra(db,sender,text,mentions):
    from database import get_user
    from config import CURRENCY
    import random
    t=text.strip(); u=get_user(db,sender)
    if t=="توب":
        users=sorted(db["users"].items(), key=lambda x: x[1].get("balance",0), reverse=True)[:10]
        lines=[f"{i+1}. @{num} - {d['balance']} {CURRENCY}" for i,(num,d) in enumerate(users)]
        return "🏆 أغنى 10:\n"+"\n".join(lines) if lines else "لا يوجد لاعبين"
    if t.startswith("رهان"):
        parts=t.split()
        if len(parts)>=2:
            try: amount=int(parts[1])
            except: return "اكتب: رهان 50"
            if u["balance"]<amount: return "❌ فلوسك لا تكفي"
            if random.choice([True,False]):
                u["balance"]+=amount; u["xp"]+=10
                return f"🎉 كسبت الرهان! +{amount} {CURRENCY}"
            else:
                u["balance"]-=amount
                return f"😢 خسرت الرهان! -{amount} {CURRENCY}"
        return "اكتب: رهان 50"
    if t.startswith("هدية") and mentions:
        parts=t.split()
        if len(parts)>=3:
            item=parts[1]; target=mentions[0]
            if item in u.get("bag",[]):
                u["bag"].remove(item)
                get_user(db,target)["bag"].append(item)
                return f"🎁 أهديت {item} إلى @{target}"
            return "❌ الصنف مش في شنطتك"
        return "اكتب: هدية سيف @منشن"
    return None
