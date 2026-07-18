<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Closure — کاربرد در Factory Functions و Module Pattern" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Closure** (بستار) وقتی ایجاد می‌شود که یک تابع به متغیرهای خارج از محدوده (scope) خودش — از یک تابع بیرونی — دسترسی داشته باشد، حتی پس از اتمام اجرای تابع بیرونی. این «بسته شدن» محیط لغوی (Lexical Environment) در زمان تعریف تابع داخلی اتفاق می‌افتد، نه زمان اجرا.

‏### هسته در سطح موتور (V8)

‏در سطح پیاده‌سازی V8:

‏1. هر تابع یک [[Scope]] internal slot دارد که اشاره‌گر به **Lexical Environment** زمان تعریف را نگه می‌دارد.
‏2. وقتی تابع بیرونی فراخوانی می‌شود، V8 یک **Execution Context** برای آن می‌سازد که شامل **VariableEnvironment** (برای var) و **LexicalEnvironment** (برای let/const) است.
‏3. اگر تابع داخلی به متغیرهای تابع بیرونی reference داشته باشد، V8 آن متغیرها را در **ScopeInfo** ذخیره کرده و در **ContextExtension** قرار می‌دهد — این یعنی متغیر به Heap منتقل می‌شود (نه Stack)، چون پس از pop شدن Execution Context باید زنده بماند.
‏4. DevTools در Memory tab این Closure objects را به صورت `[[Scopes]]` در waterfall نمایش می‌دهد.

</div>

```javascript
function outer() {
  let x = 10;          // → Heap allocation چون inner به آن reference دارد
  function inner() {
    debugger;           // Scopes panel: Closure (outer) { x: 10 }
    return x++;
  }
  return inner;
}
const fn = outer();
console.log(fn()); // 10
console.log(fn()); // 11
```

<div dir="rtl">

‏**نکته ظریف:** اگر تابع داخلی فقط متغیرهای global را بخواند و هیچ متغیری از تابع بیرونی reference نکند، Closure تشکیل **نمی‌شود** (V8 آن را بهینه می‌کند). این را با چک کردن `[[Scopes]]` در DevTools می‌توان دید.

</div>

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏به عنوان Senior، Closure را نه صرفاً به خاطر زیبایی، بلکه وقتی Trade-offها قابل قبول است انتخاب می‌کنم.

</div>

<div dir="ltr">

| **معیار** | **مزیت Closure** | **هزینه / Trade-off** |
|-----------|------------------|----------------------|
| **Memory** | داده‌ها تا زمانی که function reference زنده است در Heap می‌مانند — برای cache مؤثر است | اگر اشتباه استفاده شود **Memory Leak** قطعی است (closure scope در Heap نگه داشته می‌شود حتی اگر فقط یک متغیر از ۵۰ متغیر نیاز باشد) |
| **Encapsulation** | تنها مکانیسم واقعی Private State در JS قبل از ES2022 Class Fields | با `WeakMap` یا `Symbol` هم می‌شود، اما Closure ساده‌ترین راه است |
| **Performance** | دسترسی به متغیرهای Closure یک Property Lookup اضافه دارد (کمتر از ۱٪ تأثیر) | Closure scope زنجیره‌ای (nested closures) می‌تواند Lookup را O(n) کند — حداکثر عمق توصیه‌شده: ۳ سطح |
| **Bundle Size** | بدون وابستگی خارجی | Closure scope تمام متغیرهای reference شده را در خروجی minified نگه می‌دارد (قابل Tree-shaking نیست) |
| **Debugging** | DevTools دقیقاً Closure scope را نمایش می‌دهد | خطاهای ناشی از stale closure (مثل `var` در حلقه) سخت‌ترین خطاها برای Debug هستند |

</div>

<div dir="rtl">

‏### قانون سرانگشتی

‏- Closure برای **State Encapsulation** (مثل `useState`، `useRef`) عالی است
‏- Closure برای **Data-heavy scope** (مثل آرایه بزرگ در تابع بیرونی) مناسب نیست — بهتر است آن scope را Destructure کنید فقط متغیرهای مورد نیاز را نگه دارید

</div>

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: Memory Leak پنهان در Module Pattern با Singleton

‏یک سناریوی واقعی در پروژه‌ای با معماری Micro-frontend (هر MF یک Module Pattern داشت):

‏تیمی یک **Shared Registry** با Module Pattern ساخته بود که تمام پلاگین‌ها را کش می‌کرد:

</div>

```javascript
const PluginRegistry = (() => {
  const registry = new Map();   // ← این هیچوقت GC نمی‌شد
  const metadata = new Map();   // ← این هم
  
  return {
    register(name, plugin) {
      registry.set(name, plugin);
      metadata.set(name, { registeredAt: Date.now(), status: 'active' });
      return () => unregister(name); // ← closure روی name, registry, metadata
    }
  };
})();
```

<div dir="rtl">

‏**مشکل:** با هر بار Hot Reload در توسعه، پلاگین‌های قدیمی Registry را رها نمی‌کردند چون `metadata` سلول‌های قبلی را نگه می‌داشت و هیچ GC‌ای نمی‌توانست آن‌ها را جمع کند — حافظه هر ۳۰ دقیقه ۲۰۰MB افزایش می‌یافت.

‏**راه‌حل Production:**

</div>
<div dir="ltr">

```javascript
// x از WeakMap به جای Map استفاده کردیم
const PluginRegistry = (() => {
  // x حالا اگر plugin از بین برود، WeakMap اجازه GC می‌دهد
  const registry = new WeakMap();   // کلید: plugin object, مقدار: metadata
  /* x و metadata مستقیماً در registry نیست */
  
  return {
    register(name, plugin) {
      const unregisterFn = () => {
        // x فقط reference plugin از registry پاک می‌شود
        registry.delete(plugin);
      };
      registry.set(plugin, { name, registeredAt: Date.now() });
      return unregisterFn; // closure روی plugin و registry — نه metadata
    }
  };
})();
```
</div>

<div dir="rtl">

‏**نکته کلیدی که Junior نمی‌داند:** 
‏۱. Closure تمام متغیرهای تابع بیرونی را در Scope خود نگه می‌دارد، **حتی متغیرهایی که استفاده نمی‌شوند** — این «Shared Scope» معضل است.
‏۲. راه‌حل: **Function-scoped variable** را با **Block-scoped** (let/const) بشکنید — فقط متغیرهایی که نیاز دارید در یک بلاک جدا تعریف کنید.

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏اگر React/Next.js نبود و مجبور بودم از **Vue.js 3 + Pinia** یا **Angular 18 + Signals** استفاده کنم:

‏### مقایسه جایگزین‌ها

</div>

<div dir="ltr">

| **نیاز** | **راه‌حل با Closure (JS ساده)** | **راه‌حل در Vue/Angular** |
|----------|----------------------------------|---------------------------|
| Private State در کامپوننت | `useRef` یا `useMemo` با Closure | Vue: `ref()` داخل `<script setup>` (ماکروی کامپایلر) — Angular: `signal()` |
| Event Handler با state تازه | `useCallback` با Closure | Vue: template ref — Angular: `(click)` binding |
| Shared State بین کامپوننت‌ها | Context + Closure | Vue: Pinia (state = `reactive()` پشت پرده) — Angular: Signals + NgRx |
| Custom Hook / Composition | Closure + Hooks | Vue: Composables (دقیقاً closure) — Angular: Injectable services |

</div>

<div dir="rtl">

‏### Next.js vs Vue.js — کدام برتری دارد؟

‏برای این نیاز خاص **(State Encapsulation با Closure)**، Next.js و Vue.js تفاوت زیادی ندارند چون هر دو از Closure استفاده می‌کنند. اما:

‏- **Next.js** نیاز به `'use client'` و Hooks دارد — یک Indirection اضافه
‏- **Vue 3 Composition API** Closure را مستقیماً در `setup()` استفاده می‌کند — transparent
‏- **Angular Signals** از `WeakRef` و `ReactiveNode` در پشت پرده استفاده می‌کند — abstraction بالا

‏**نتیجه:** برای State Encapsulation ساده، Vue 3 با Composition API **شفاف‌ترین** است. اما برای Enterprise Scale با Server Components، Next.js به خاطر **RSC Payload** و **Serialization** برتری دارد چون Closure در Server Components مجاز نیست و Developer را مجبور به فکر کردن درباره مرز سرور/کلاینت می‌کند — که در مقیاس بزرگ از Leak جلوگیری می‌کند.

</div>

---

<div dir="rtl">

‏> **پس از تحویل این مبحث، منتظر تأیید من بمان. پس از تأیید، طبق دستورات قسمت Automation Instructions پیش برو و سپس متوقف شو.**

</div>