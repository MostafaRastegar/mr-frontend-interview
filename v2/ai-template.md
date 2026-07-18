# ⚙️ Automation Instructions (DO NOT DELETE)
>
> This block controls session behavior. Read fully before each run.
>
> ## Session Bootstrap
> 1. Read `v2/progress.md` — find first `- [ ]` line (unchecked topic).
> 2. Determine context from progress.md section headers:
>    - Which **list**: فهرست ۱ (JS Core), ۲ (React/Next), ۳ (Architecture)
>    - Which **priority**: ⭐⭐⭐ (ضروری), ⭐⭐ (مهم), ⭐ (پیشرفته)
>    - Topic title (same as progress.md entry)
> 3. Read `v2/content/INDEX.md` — append row to this when done.
> 4. Read this template (below) — it defines the 4-level output structure.
>
> ## Workflow Rules
> - **Generate ONE topic per session** — find first unchecked, produce full answer, STOP.
> - After presenting the answer → **wait for user approval**.
> - On approval:
>   ```
>   # Mark checkbox in progress.md:
>   #   Find first "- [ ]" line, replace with "- [x]"
>   LINE=$(grep -n '^- \[ \]' v2/progress.md | head -1 | cut -d: -f1)
>   sed -i "${LINE}s/- \[ \]/- [x]/" v2/progress.md
>   #
>   # Prepend row to INDEX.md (insert after header row line 3):
>   date_str=$(date +%Y-%m-%d)
>   sed -i "3i| $date_str | {topic} | {list} | v2/content/{slug}.md |" v2/content/INDEX.md
>   ```
> - On rejection → revise based on feedback, re-present, wait again.
> - After marked done → **STOP**. No next topic. User will re-run for next.
>
> ## Content File
> - Save full answer to `v2/content/{topic-slug}.md`
> - topic-slug: lowercase, Farsi words transliterated or English shorthand
> 
> ## RTL Formatting Rules (CRITICAL — Persian content)
> - **All Persian text** must be wrapped in `<div dir="rtl">` ... `</div>`
> - **Every single non-empty line** inside `<div dir="rtl">` MUST start with RLM character (U+200F, `\u200F`).  
>   This includes headings, bullet items, numbered items, blockquotes, separators — literally every line.
> - **Code blocks, diagrams, tables, ASCII art** must be wrapped in `<div dir="ltr">` ... `</div>` (no RLM needed there).  
>   NEVER put code blocks (triple backtick) inside a RTL div — always close the RTL div first, place the code block, then re-open a new RTL div.
> - **Mixed line** (e.g. `۱. **Call Stack**: یک LIFO stack...`): still needs RLM prefix because the line is in a RTL context.
> - **Inline code** inside Persian paragraphs is fine — the RTL div handles it.
> - **Blank line after every opening `<div dir="rtl">` tag** — content must NOT be glued to the tag. Always:  
>   ```  
>   <div dir="rtl">  
>                            ← blank line  
>   ‏متن فارسی اینجا...
>   ```  
>   Same for `<div dir="ltr">` — blank line after opening tag.
> - After writing the file, run this post-processing script to auto-add RLM:
>   ```bash
>   node -e "
>   const fs=require('fs'), R='\u200F';
>   let c=fs.readFileSync('v2/content/{slug}.md','utf8');
>   c=c.replace(/<div dir=\"rtl\">([\s\S]*?)<\/div>/g,(m,i)=>{
>     return '<div dir=\"rtl\">'+i.split('\n').map(l=>l.trim()&&!l.startsWith(R)?R+l:l).join('\n')+'</div>'
>   });
>   fs.writeFileSync('v2/content/{slug}.md',c,'utf8');
>   "
>   ```
>   Replace `{slug}` with actual filename. This ensures no line is missed.
>
> ## Next-Topic Detection (Quick Reference)
> ```bash
> grep -n '^- \[ \]' v2/progress.md | head -3
> ```
> This shows next 3 pending topics. First match = current target.

---

شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد. 

لطفاً سوال "{TOPIC_NAME}" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

**سطح ۱ (تعریف و هسته فنی):**
تعریف دقیق و هسته‌ی این مفهوم در اکوسیستم React و Next.js (نسخه ۱۴ به بالا) چیست؟ لطفاً از ذکر تعاریف کلیشه‌ای و سطحی خودداری کن و به جزئیات پیاده‌سازی در سطح V8/موتور مرورگر یا معماری Fiber اشاره کن.

**سطح ۲ (معیارهای انتخاب و تصمیم‌گیری - مهم‌ترین بخش):**
به عنوان یک Senior، چه معیارهایی را برای استفاده یا عدم استفاده از این رویکرد در نظر می‌گیرم؟ لطفاً این معیارها را در قالب یک جدول مقایسه‌ای (مثلاً: Trade-offs، تأثیر بر SEO، تأثیر بر TTI و TBT، هزینه‌ی سرور، و تجربه‌ی توسعه‌دهنده) ارائه بده.

**سطح ۳ (چالش واقعی در پروژه‌های بزرگ):**
دقیقاً در یک پروژه‌ی واقعی با معماری Micro-frontend یا Monorepo، چه چالش مشخصی سر راه من قرار می‌گیرد که یک توسعه‌دهنده‌ی Junior متوجه آن نمی‌شود؟ لطفاً چالش را سناریو محور توضیح بده و راه‌حل نهایی‌ای که در Production به کار برده‌ای را به صورت کد-مانند (Code snippet) یا الگوی معماری ارائه کن.

**سطح ۴ (عدم استفاده و راه‌کار جایگزین در اکوسیستم):**
اگر React/Next.js در اختیار من نبود یا مجبور بودم از Vite یا Angular استفاده کنم، چگونه این نیاز را برطرف می‌کردم؟ مقایسه‌ای بین پیچیدگی پیاده‌سازی در Next.js با راه‌کار جایگزین (مثلاً استفاده از SWR + Express.js) انجام بده و دلیل برتری Next.js را در این مورد خاص توضیح بده.

---

> **پس از تحویل این مبحث، منتظر تأیید من بمان. پس از تأیید،‌ طبق دستورات قسمت Automation Instructions پیش برو و سپس متوقف شو.**
