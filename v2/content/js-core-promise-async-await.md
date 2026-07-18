<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Promise و Async/Await (مدیریت خطا، `Promise.all`, `Promise.race`, `Promise.allSettled`)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏Promise آبجکتی است که نتیجه‌ی یک عملیات asynchronous را در آینده نمایش می‌دهد — سه حالت دارد: `pending`, `fulfilled`, `rejected`. یک Promise «thenable» است (یکبار مصرف) و پس از settle شدن، دیگر تغییر نمی‌کند.

‏Async/Await شکر syntactical روی Promise است — هر async function یک Promise برمی‌گرداند. Await جریان اجرا را تا settle شدن Promise متوقف می‌کند (بدون blocking ترد).

</div>

```javascript
// x Internal state machine — ساده‌شده
const PENDING = 0, FULFILLED = 1, REJECTED = 2;

class SimplePromise {
  #state = PENDING;
  #value = undefined;
  #queue = [];

  constructor(executor) {
    const resolve = (value) => this.#settle(FULFILLED, value);
    const reject = (reason) => this.#settle(REJECTED, reason);
    try { executor(resolve, reject); } catch (e) { reject(e); }
  }

  #settle(state, value) {
    if (this.#state !== PENDING) return; // settled once
    this.#state = state;
    this.#value = value;
    this.#queue.forEach(([onF, onR]) => {
      queueMicrotask(() => this.#handle(onF, onR));
    });
    this.#queue = [];
  }

  #handle(onF, onR) {
    const handler = this.#state === FULFILLED ? onF : onR;
    if (typeof handler !== 'function') return;
    try { handler(this.#value); } catch (e) { /* unhandled */ }
  }

  then(onF, onR) {
    return new SimplePromise((resolve, reject) => {
      const wrapF = (v) => { try { resolve(onF ? onF(v) : v); } catch(e) { reject(e); } };
      const wrapR = (r) => { try { reject(onR ? onR(r) : r); } catch(e) { reject(e); } };
      if (this.#state === PENDING) {
        this.#queue.push([wrapF, wrapR]);
      } else {
        queueMicrotask(() => this.#handle(wrapF, wrapR));
      }
    });
  }

  catch(onR) { return this.then(undefined, onR); }
  finally(fn) { return this.then(
    v => { fn(); return v; },
    r => { fn(); throw r; }
  ); }
}

// Microtask ordering
console.log('1: sync');
Promise.resolve().then(() => console.log('2: microtask'));
setTimeout(() => console.log('3: macrotask'), 0);
console.log('4: sync');
// output: 1 → 4 → 2 → 3
```

<div dir="rtl">

‏حافظه: Microtask Queue بالاتر از Macrotask Queue اولویت دارد — Event Loop بعد از هر Macrotask، کل Microtask Queue را تخلیه می‌کند. به همین دلیل `Promise.resolve().then(...)` قبل از `setTimeout(0)` اجرا می‌شود.

</div>

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏| روش | کاربرد | مزیت | محدودیت |
‏|-----|--------|------|---------|
‏| `Promise.all` | چند درخواست مستقل همزمان | Fail-fast — اگر یکی rejected شود، همه abort می‌شوند | نمی‌خواهی Fail-fast — بقیه نتایج را از دست می‌دهی |
‏| `Promise.allSettled` | همه‌ی نتایج را می‌خواهی (مهم نیست fail شوند) | هیچ reject propagation ندارد | همیشه صبر می‌کند تا همه تمام شوند |
‏| `Promise.race` | timeout روی یک درخواست | اولین settled result (resolve or reject) را می‌دهد | ابهام — نمی‌دانی کدام برنده شد |
‏| `Promise.any` | اولین resolve موفق | اولین success را برمی‌گرداند، ignore reject | اگر همه rejected شوند AggregateError می‌دهد |
‏| Sequential `await` | وابستگی ترتیبی (نتیجه d به c و c به b وابسته است) | ساده، خوانا | کندتر از parallel |

</div>

```javascript
// Production patterns
async function fetchWithTimeout(url, ms = 5000) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), ms);
  try {
    return await fetch(url, { signal: controller.signal });
  } finally {
    clearTimeout(timer);
  }
}

// Parallel with partial fallback
async function fetchAllWithFallback(urls) {
  const results = await Promise.allSettled(urls.map(fetch));
  return results.map((r, i) =>
    r.status === 'fulfilled'
      ? r.value
      : { url: urls[i], error: r.reason, fallback: true }
  );
}

// Sequential with dependency
async function deepFetch(userId) {
  const user    = await api.getUser(userId);
  const posts   = await api.getPosts(user.id);
  const details = await Promise.all(
    posts.map(p => api.getPostDetails(p.id))
  );
  return { user, posts, details };
}
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏**سناریو:** یک micro-frontend shell که ۴ remote app را همزمان لود می‌کند. هر app فایل‌های JS خودش را از CDN می‌گیرد. اگر `Promise.allSettled` نزنی و یک app از کار بیفتد (CDN outage)، کل shell بارگذاری نمی‌شود.

‏**چالش:** `Promise.all` در یک micro-frontend host باعث می‌شود اگر remote A 503 بدهد، remote B و C و D هم لود نشوند — وبسایت کاملاً سفید می‌ماند.

‏**راه‌حل:** `Promise.allSettled` + fallback UI per remote + health check recovery.

</div>

```javascript
// x Micro-frontend loader با resilience
const REMOTE_APPS = [
  { name: 'catalog', url: '/remote/catalog.js' },
  { name: 'cart',    url: '/remote/cart.js' },
  { name: 'checkout',url: '/remote/checkout.js' },
  { name: 'profile', url: '/remote/profile.js' },
];

async function loadRemotes() {
  const loaded = await Promise.allSettled(
    REMOTE_APPS.map(({ name, url }) =>
      import(/* webpackIgnore: true */ url)
        .then(mod => ({ name, mod }))
        .catch(err => {
          console.error(`[Shell] ${name} failed:`, err);
          return null; // graceful degradation
        })
    )
  );

  const successes = loaded
    .filter(r => r.status === 'fulfilled' && r.value !== null)
    .map(r => r.value);

  const failures = REMOTE_APPS.length - successes.length;
  if (failures > 0) {
    // Sentry/custom error reporting
    reportError(`Micro-frontend: ${failures}/${REMOTE_APPS.length} failed`);
  }

  return {
    apps: successes,
    fallback: failures > 0,
    retry: (name) => loadSingleRemote(name), // lazy retry بعد از بهبود CDN
  };
}

// Health check periodic recovery
setInterval(async () => {
  const failed = REMOTE_APPS.filter(a => !isLoaded(a.name));
  if (failed.length === 0) return;
  await loadFailedRemotes(failed);
  updateShellRouting(); // به‌روزرسانی navigation
}, 30000);
```

<div dir="rtl">

‏**نکته Performance:** `Promise.all` با ۱۰۰ درخواست همزمان می‌تواند `connection pool` را اشغال کند. در production از `p-limit` یا batch concurrency control استفاده کن — مرورگر معمولاً ۶ connection concurrent per domain دارد.

</div>

```javascript
// Concurrency limiter
async function pMap(items, fn, concurrency = 6) {
  const results = [];
  const executing = new Set();

  for (const [index, item] of items.entries()) {
    const p = fn(item).then(r => results[index] = r);
    executing.add(p);
    p.finally(() => executing.delete(p));

    if (executing.size >= concurrency) {
      await Promise.race(executing);
    }
  }

  await Promise.all(executing);
  return results;
}
```

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏بدون Promise/Async (ES5, callbacks):

</div>

| رویکرد | روش | معایب |
|--------|-----|-------|
| Callback hell | تو در تو, error-first pattern | nesting, no error propagation, inversion of control |
| EventEmitter | `.on('data')` / `.on('error')` | scattered logic, memory leak (unsubscribe) |
| Observable (RxJS) | `.pipe(map, filter, mergeMap)` | heavy dependency, overkill for simple cases |
| Generators + co | `function*` + `yield` → wrapper | deprecated, replaced by async/await |
| Web Workers + postMessage | پیام‌محور | فقط cross-thread, نه general purpose |

<div dir="rtl">

‏**مقایسه Next.js vs Express.js:** در Next.js Server Components، تمام data fetching **به صورت Promise در Server Side انجام می‌شود** — نیازی به `useEffect` + fetch نیست. Next.js از React cache استفاده می‌کند تا Promiseهای تکراری را deduplicate کند. در Express.js، Promiseها معمولاً در route handlerها با `async(req, res, next)` مدیریت می‌شوند — خبری از deduplication پیش‌فرض نیست و باید خودت middleware بزنی (مثلاً `lru-cache`).

</div>

<div dir="rtl">

‏> **نهایی:** Promise و Async/Await پایه‌ی تمام عملیات async در JS مدرن هستند. نکته‌ی کلیدی Senior: تفاوت میان `Promise.all` (fail-fast) و `Promise.allSettled` (همه نتایج) را در سناریوهای resilience بدان. همچنین ترتیب اجرای microtask (Promise) vs macrotask (setTimeout, I/O) را دقیقاً توضیح بده — این سوال ثابت مصاحبه‌های Senior است.

</div>
