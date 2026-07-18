<div dir="rtl">

‏# Event Loop معماری (Call Stack, Microtask Queue vs Macrotask Queue)

‏> **تولیدشده بر اساس تمپلیت ۴ سطحی | مختص Senior ۶+ سال | تاریخ: 2026-07-18**

‏---

## ‏سطح ۱ (تعریف و هسته فنی)

‏Event Loop در JavaScript یک **حلقه‌ی بی‌پایان** در موتور (V8 برای Chrome/Node.js، SpiderMonkey برای Firefox) است که وظیفه‌اش هماهنگی بین **Call Stack**، **Task Queue (Macrotask)** و **Microtask Queue** می‌باشد.

### ‏هسته‌ی معماری در سطح موتور:

‏۱. **Call Stack**: یک LIFO stack از execution contexts. V8 هر بار یک تابع را push می‌کند، اجرا می‌کند، pop می‌کند. تا وقتی Call Stack خالی نشود، V8 به هیچ Queueای نگاه نمی‌کند.

‏۲. **Microtask Queue** (معروف به Job Queue در ES2015+): اولویت بالاتر. هر تیک Event Loop، *قبل از* رفتن به Macrotask بعدی، **تمام** Microtaskهای صف را خالی می‌کند. این شامل:
‏   - `Promise.then/catch/finally` callbackها
‏   - `queueMicrotask()`
‏   - `MutationObserver` callbackها

‏۳. **Macrotask Queue** (Task Queue در WHATWG): شامل:
‏   - `setTimeout` / `setInterval`
‏   - `setImmediate` (Node.js)
‏   - I/O taskها (fs.readFile, network request)
‏   - UI rendering events (requestAnimationFrame technically یک task است، ولی با اولویت خاص)
‏   - Event handlers (click, keydown, etc.)

‏۴. **Animation Frame Callback Queue**: `requestAnimationFrame` یک صف مجزا دارد که بین رندر مرورگر اجرا می‌شود (قبل از repaint).

### ‏ترتیب دقیق اجرا (الگوریتم مرورگر):

</div>

<div dir="ltr">

```
┌──────────────────────────────────────┐
│ ۱. اجرای یک Macrotask از Task Queue    │
│    (مثلاً اسکریپت اولیه، کلیک، setTimeout)│
└──────────┬───────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│ ۲. بررسی Microtask Queue             │
│    - اجرای تمام Microtaskها تا خالی  │
│    - Microtaskهای جدید هم اجرا شوند! │
│    (به همین دلیل `Promise.resolve()  │
│     .then(...)` بی‌نهایت نکش می‌دهد) │
└──────────┬───────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│ ۳. rAF callbackها (قبل از رندر)       │
└──────────┬───────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│ ۴. رندر و Paint (style, layout, paint)│
└──────────┬───────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│ ۵. Idle callback (requestIdleCallback)│
└──────────┬───────────────────────────┘
           ↓
           ══ تکرار از مرحله ۱ ══
```

</div>

<div dir="ltr">

### مثال بحرانی:

```javascript
console.log('1 (call stack)');

setTimeout(() => console.log('2 (macrotask)'), 0);

Promise.resolve()
  .then(() => console.log('3 (microtask)'))
  .then(() => console.log('4 (microtask chain)'));

requestAnimationFrame(() => console.log('5 (rAF)'));

queueMicrotask(() => console.log('6 (microtask direct)'));

console.log('7 (call stack)');

// x خروجی: 1 → 7 → 3 → 4 → 6 → 5 → 2
```

نکته: rAF (۵) قبل از setTimeout (۲) اجرا می‌شود چون rAF در مرحله‌ی ۳ از Event Loop است، در حالی که setTimeout محصول یک Macrotask جدید است که به مرحله‌ی ۱ برمی‌گردد.

</div>

---

<div dir="rtl">

## ‏سطح ۲ (معیارهای انتخاب و تصمیم‌گیری)

‏این بخش بیشتر از جنس **فهم اشکال‌زدایی و بهینه‌سازی** است تا انتخاب یک رویکرد.

### ‏جدول Trade-off:

</div>

<div dir="ltr">

| معیار | اولویت Microtask (Promise) | اولویت Macrotask (setTimeout) | اولویت rAF |
|-------|---------------------------|-------------------------------|------------|
| **زمان اجرا** | بلافاصله بعد از Call Stack خالی | با تأخیر حداقل ۴ms (طبق spec برای setTimeout تودرتو) | قبل از Paint بعدی (~16.6ms) |
| **مسدود کردن UI** | اگر Microtask بی‌نهایت بزند، مرورگر قفل می‌کند | مسدود نمی‌کند (مگر اجرای طولانی) | مسدود نمی‌کند (اولویت رندر) |
| **ترتیب تضمینی** | FIFO دقیق در همان تیک Event Loop | FIFO در تیک بعدی (غیرقابل پیش‌بینی با سایر Macrotaskها) | FIFO در تیک رندر بعدی |
| **تأثیر بر TTI** | می‌تواند TTI را به تعویق بیندازد (blocking) | به تأخیر می‌اندازد ولی غیرمسدودکننده | فقط روی رندر تأثیر دارد |
| **معماری Micro-frontend** | مشکل: اگر دو میکروفرانت Promise یکسان داشته باشند، یکی می‌تواند Microtaskهای دیگری را قورت بدهد | ایمن‌تر برای ارتباط بین micro-frontendها | مناسب برای انیمیشن هماهنگ بین micro-frontendها |
| **تجربه‌ی توسعه‌دهنده** | ساده و شهودی (async/await) | callback + مدیریت time پیچیده‌تر | محدود به انیمیشن و visual update |

</div>

<div dir="rtl">

### ‏چه موقع از Microtask استفاده کنیم:
‏- **Promise chain** برای وابستگی داده
‏- **State update** بعد از fetch در React (Batch update)
‏- **queueMicrotask** برای عملیات‌هایی که باید قبل از UI اجرا شوند

### ‏چه موقع از Macrotask استفاده کنیم:
‏- **Heavy computation** → شکستن به chunkها با `setTimeout(0)` برای آزاد کردن UI
‏- **Debounce/Throttle** در input handler
‏- **Scheduling** با delay مشخص

‏---

## ‏سطح ۳ (چالش واقعی در پروژه‌های بزرگ)

### ‏سناریو: Event Loop Blocking در Monorepo با ۳۰+ تیم

‏**معماری**: یک SPA با Module Federation (تیم‌های مجزا روی micro-frontendهای مختلف کار می‌کنند). هر micro-frontend در رخداد `app:route-change` یک Promise chain برای initialize کردن ماژول خودش اجرا می‌کند.

‏**مشکل**: تیم Payment یک `Promise.all(...)` با ۵۰ endpoint می‌زند. هر resolve یک `.then` زنجیره‌ای می‌سازد که مجموعاً ۵۰ microtask پشت سر هم. تیم Profile همزمان یک `useEffect` با `queueMicrotask` برای sync کردن state دارد. نتیجه: **Microtask Queue هرگز خالی نمی‌شود** و مرورگر برای ۳-۴ ثانیه قفل می‌کند (UI frozen, scroll نمی‌خورد).

‏**چرا Junior متوجه نمی‌شود**: فکر می‌کند asynchronous یعنی non-blocking. نمی‌داند که Microtaskها بین دو رندر اجرا می‌شوند و اگر بی‌نهایت باشند، رندر هرگز اتفاق نمی‌افتد.

‏**راه‌حل Production: استفاده از Scheduler.yield (یا postTask)**

</div>

<div dir="ltr">

```typescript
// utils/scheduler.ts
// x راه‌حل: شکستن Promise chain طولانی با yielding به Event Loop

export async function yieldToMainThread(deadline?: number) {
  // x اگر scheduler.yield در دسترس است (Chrome 115+)
  if ('scheduler' in window && 'yield' in (window as any).scheduler) {
    return (window as any).scheduler.yield();
  }
  
  // x Fallback: ترکیب setTimeout + postMessage برای اولویت بالاتر از setTimeout
  return new Promise<void>(resolve => {
    if (deadline && deadline > 0) {
      setTimeout(resolve, deadline);
    } else {
      // x postMessage از setTimeout سریع‌تر است (قبل از رندر)
      const channel = new MessageChannel();
      channel.port1.onmessage = () => resolve();
      channel.port2.postMessage(null);
    }
  });
}

// x استفاده در initialization chain
async function initializePaymentModule() {
  const chunkSize = 5;
  const endpoints = [...]; // 50 endpoint
  
  for (let i = 0; i < endpoints.length; i += chunkSize) {
    const chunk = endpoints.slice(i, i + chunkSize);
    await Promise.all(chunk.map(fetch));
    
    // x بعد از هر ۵ تا، به Event Loop yield کن
    await yieldToMainThread();
  }
}
```

</div>

<div dir="rtl">

‏**معماری نهایی در پروژه:**
</div>

<div dir="ltr">

```
[Root App]
    ├── [Scheduler Orchestrator] ← کنترل اولویت Taskها
    │    ├── Priority: HIGH (user interaction) → بلافاصله اجرا شود
    │    ├── Priority: MEDIUM (data fetch) → yield بعد از Microtask
    │    └── Priority: LOW (analytics, logging) → با requestIdleCallback
    │
    ├── [Micro-frontend A (Payment)]
    │    └── از Scheduler Orchestrator برای initialization استفاده می‌کند
    │
    ├── [Micro-frontend B (Profile)]
    │    └── از Scheduler Orchestrator استفاده می‌کند
    │
    └── [Monitoring Service]
         └── هر ۵ ثانیه Event Loop delay را اندازه می‌گیرد
```

</div>

<div dir="rtl">

‏**سناریوی جایگزین در Next.js**: اگر از Server Components استفاده کنید، این مشکل وجود ندارد چون Promise chainها سمت سرور اجرا می‌شوند و هر کدام در یک Worker Pool مجزا (نه در Event Loop مرورگر).

‏---

## ‏سطح ۴ (عدم استفاده و راه‌کار جایگزین در اکوسیستم)

### ‏اگر Next.js نبود: پیاده‌سازی با Vite + Express.js + setTimeout scheduling

‏در Next.js، Server Components + `async/await` مستقیماً روی سرور اجرا می‌شوند و خروجی HTML/stream به کلاینت می‌رود. Event Loop کلاینت آزاد است.

‏بدون Next.js (مثلاً Vite + Express.js):

</div>

<div dir="ltr">

```typescript
// Express.js server: manual async handler
app.get('/api/data', async (req, res) => {
  const data = await db.query('SELECT * FROM large_table'); // blocking
  res.json(data);
});
// x مشکل: Event Loop Node.js مشغول DB query می‌شود.
// x راه‌حل: Worker Thread یا partitioning query
```

**راه‌کار جایگزین با Vite + Express.js + Microtask Manager**:

```typescript
// server/scheduler.ts
import { Worker } from 'worker_threads';
import { EventEmitter } from 'events';

export class TaskScheduler extends EventEmitter {
  private worker: Worker;
  
  constructor() {
    super();
    this.worker = new Worker('./workers/heavy-task.js', {
      workerData: { /* config */ }
    });
  }
  
  schedule(task: Task, priority: 'high' | 'low'): void {
    // x low priority → از Macrotask (setImmediate) استفاده کن
    if (priority === 'low') {
      setImmediate(() => this.worker.postMessage(task));
    } else {
      // x high priority → بلافاصله در Worker (نه Event Loop)
      this.worker.postMessage(task);
    }
  }
}
```

</div>

<div dir="rtl">

### مقایسه Next.js vs Vite + Express.js:

</div>

<div dir="ltr">

| معیار | Next.js (Server Components) | Vite + Express.js + Scheduler |
|-------|------------------------------|-------------------------------|
| **مدیریت Event Loop سمت سرور** | خودکار (هر درخواست در یک worker pool مجزا) | دستی (نیاز به Worker Thread + Scheduler) |
| **Streaming** | Built-in (Suspense + streaming SSR) | نیاز به manual pipe با Readable Stream |
| **Microtask Queue سمت کلاینت** | کمترین (HTML از سرور می‌آید) | زیاد (hydration + data fetching با useEffect) |
| **پیچیدگی پیاده‌سازی** | کم (فقط async component) | زیاد (نیاز به Scheduler, Worker Pool, Memory management) |
| **قابلیت Debug** | متوسط (سرور + کلاینت) | بالا (کاملاً در کنترل شماست) |
| **انعطاف‌پذیری برای Micro-frontend** | محدود (Module Federation با Next.js سخت است) | بالا (Vite + Module Federation ساده‌تر) |

</div>

<div dir="rtl">

### ‏دلیل برتری Next.js در این مورد خاص:

‏در معماری‌های مدرن با Server Components، **بار پردازشی Event Loop از روی کلاینت برداشته می‌شود** و به سرور منتقل می‌شود. در سرور هم هر درخواست در یک context مجزا اجرا می‌شود (isolated by design). این یعنی مشکل Microtask Queue blocking که در SPAهای کلاینت-محور رایج است، ذاتاً در Next.js 14+ وجود ندارد — مگر اینکه Client Component بنویسید که دسترسی به Event Loop مرورگر داشته باشد.

‏**نتیجه**: Next.js برنده است چون:
‏1. Event Loop client-side را آزاد نگه می‌دارد
‏2. Server Components از blocking microtaskها جلوگیری می‌کنند
‏3. Built-in streaming بدون نیاز به scheduler سفارشی
‏4. اما: اگر معماری Micro-frontend دارید و همه Client Component هستند → فرقی با Vite ندارد → باید خودتان Scheduler بنویسید

‏---

‏> **پس از تحویل این مبحث، منتظر تأیید من بمان. پس از تأیید،‌ طبق دستورات قسمت Automation Instructions پیش برو و سپس متوقف شو.**

</div>