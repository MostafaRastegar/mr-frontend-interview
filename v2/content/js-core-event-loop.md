
آشنایی با Event Loop (Call Stack, Microtask Queue, Macrotask Queue)
مناسب برای همه سطوح | تاریخ: 2026-07-18

Event Loop چیست؟
Event Loop در JavaScript یک حلقه‌ی بی‌پایان در موتور (مثل V8 در Chrome) است که کارش هماهنگی بین Call Stack، Microtask Queue و Macrotask Queue است.

بخش‌های اصلی:
۱. Call Stack: یک پشته (Stack) از توابع در حال اجرا. JavaScript یک‌خطی است — هر بار فقط یک تابع اجرا می‌شود. تا وقتی Call Stack خالی نشود، به صف‌های دیگر سر نمی‌زند.

۲. Microtask Queue (اولویت بالا): کارهای کوچکی که باید همین الان بعد از اتمام تابع فعلی اجرا شوند. شامل:

Promise.then/catch/finally
queueMicrotask()
MutationObserver
۳. Macrotask Queue (اولویت پایین‌تر): کارهایی که می‌توانند کمی صبر کنند. شامل:

setTimeout / setInterval
I/O مثل خواندن فایل یا درخواست شبکه
رویدادهای کاربر (کلیک، تایپ)
setImmediate (در Node.js)
۴. Animation Frame Callback Queue: requestAnimationFrame صف جداگانه‌ای دارد که قبل از رندر اجرا می‌شود.

ترتیب دقیق اجرا:
┌──────────────────────────────────────┐
│ ۱. اجرای یک Macrotask (مثلاً کلیک)     │
└──────────┬───────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│ ۲. اجرای تمام Microtaskها تا خالی شدن │
│    (Microtaskهای جدید هم اجرا شوند)   │
└──────────┬───────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│ ۳. rAF callbackها                     │
└──────────┬───────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│ ۴. رندر مرورگر (style, layout, paint) │
└──────────┬───────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│ ۵. Idle callback (requestIdleCallback)│
└──────────┬───────────────────────────┘
           ↓
           ══ از اول ══
مثال ساده:
console.log('1');

setTimeout(() => console.log('2'), 0);        // macrotask

Promise.resolve()
  .then(() => console.log('3'))               // microtask
  .then(() => console.log('4'));              // microtask

requestAnimationFrame(() => console.log('5')); // rAF

queueMicrotask(() => console.log('6'));        // microtask

console.log('7');

// خروجی: 1 → 7 → 3 → 4 → 6 → 5 → 2
توضیح خروجی: اول 1 و 7 (Call Stack). بعد همه Microtaskها (3, 4, 6). بعد rAF (5). آخر setTimeout (2).

Microtask vs Macrotask: کی از کدام استفاده کنیم؟
Microtask (Promise, queueMicrotask):
زمان اجرا: بلافاصله بعد از خالی شدن Call Stack
مزیت: سریع، به‌راحتی با async/await قابل استفاده
مشکل: اگر بی‌نهایت Microtask بسازید، مرورگر قفل می‌کند (رندر انجام نمی‌شود)
کی استفاده کنیم: وابستگی داده (Promise chain)، آپدیت state در React
Macrotask (setTimeout):
زمان اجرا: با تأخیر (حداقل ۴ms برای setTimeout تودرتو)
مزیت: غیرمسدودکننده — به مرورگر فرصت رندر می‌دهد
کی استفاده کنیم: محاسبات سنگین (با شکستن به chunkهای کوچک)، Debounce/Throttle
rAF (requestAnimationFrame):
زمان اجرا: قبل از Paint بعدی (حدود ۱۶ms)
کی استفاده کنیم: انیمیشن،视觉 به‌روزرسانی
مشکل رایج: قفل شدن مرورگر با Microtask
سناریو:
یک SPA دارید که چند تیم روی آن کار می‌کنند. یک تیم ۵۰ تا درخواست API می‌زند و هر کدام یک Promise زنجیره‌ای دارد. تیم دیگر هم queueMicrotask می‌زند. نتیجه: Microtask Queue خالی نمی‌شود و مرورگر ۳-۴ ثانیه قفل می‌کند.

چرا این مشکل پیش می‌آید؟ چون Microtaskها بین دو رندر اجرا می‌شوند. اگر مدام Microtask جدید بسازید، رندر هرگز اتفاق نمی‌افتد.

راه‌حل: به Event Loop استراحت دهید
// utils/scheduler.ts
// شکستن کارهای سنگین با استراحت دادن به Event Loop

export async function yieldToMainThread(deadline?: number) {
  // اگر مرورگر از scheduler.yield پشتیبانی می‌کند (Chrome 115+)
  if ('scheduler' in window && 'yield' in (window as any).scheduler) {
    return (window as any).scheduler.yield();
  }

  // روش جایگزین: ترکیب setTimeout + MessageChannel
  return new Promise<void>(resolve => {
    if (deadline && deadline > 0) {
      setTimeout(resolve, deadline);
    } else {
      const channel = new MessageChannel();
      channel.port1.onmessage = () => resolve();
      channel.port2.postMessage(null);
    }
  });
}

// مثال: ۵۰ تا درخواست را ۵ تایی می‌زنیم و بینشان استراحت می‌دهیم
async function initializePaymentModule() {
  const chunkSize = 5;
  const endpoints = [/* 50 endpoint */];

  for (let i = 0; i < endpoints.length; i += chunkSize) {
    const chunk = endpoints.slice(i, i + chunkSize);
    await Promise.all(chunk.map(fetch));

    // بعد از هر ۵ تا، به مرورگر فرصت رندر بده
    await yieldToMainThread();
  }
}
معماری نهایی:
[برنامه اصلی]
    ├── [Scheduler Orchestrator] ← کنترل اولویت کارها
    │    ├── HIGH (کلیک کاربر) → بلافاصله
    │    ├── MEDIUM (دریافت داده) → بعد از Microtaskها
    │    └── LOW (آمار و لاگ) → با requestIdleCallback
    │
    ├── [ماژول Payment]
    │    └── از Scheduler برای مقداردهی اولیه استفاده می‌کند
    │
    ├── [ماژول Profile]
    │    └── از Scheduler استفاده می‌کند
    │
    └── [مانیتورینگ]
         └── هر ۵ ثانیه تأخیر Event Loop را اندازه می‌گیرد
نکته مهم: در Next.js با Server Components، Promise chainها سمت سرور اجرا می‌شوند و Event Loop مرورگر آزاد می‌ماند. اما اگر همه ماژول‌های شما Client Component هستند (مثل Micro-frontendها)، فرقی با یک SPA معمولی ندارد و باید خودتان مدیریت کنید.

پس از تحویل این مبحث، منتظر تأیید من بمان. پس از تأیید، طبق دستورات قسمت Automation Instructions پیش برو و سپس متوقف شو.