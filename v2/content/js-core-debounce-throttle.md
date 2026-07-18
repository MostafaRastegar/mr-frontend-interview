<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Debouncing و Throttling (پیاده‌سازی دستی با Closure و مدیریت `this` و `event` در آن)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Debounce** — فراخوانی تابع را تا زمانی که یک مکث مشخص (delay) آخرین رویداد سپری شود، به تأخیر می‌اندازد. اگر رویداد دوباره رخ دهد، تایمر قبلی ریست می‌شود. نتیجه: تابع فقط یک بار پس از توقف کامل رویدادها اجرا می‌شود.

‏**Throttle** — فراخوانی تابع را به یک بار در هر بازه زمانی مشخص (limit) محدود می‌کند، صرف نظر از تعداد رویدادها. نتیجه: تابع با یک نرخ ثابت اجرا می‌شود.

</div>

```javascript
// x Debounce — استاندارد با leading/trailing options
function debounce(fn, delay, options = { leading: false, trailing: true }) {
  let timerId = null;
  let lastArgs = null;

  const debounced = function (...args) {
    const context = this;
    lastArgs = args;

    if (timerId === null && options.leading) {
      fn.apply(context, args);
    }

    clearTimeout(timerId);
    timerId = setTimeout(() => {
      timerId = null;
      if (options.trailing && lastArgs) {
        fn.apply(context, lastArgs);
      }
      lastArgs = null;
    }, delay);
  };

  debounced.cancel = () => {
    clearTimeout(timerId);
    timerId = null;
    lastArgs = null;
  };

  debounced.flush = () => {
    if (timerId !== null) {
      clearTimeout(timerId);
      timerId = null;
      if (lastArgs) {
        fn.apply(this, lastArgs);
        lastArgs = null;
      }
    }
  };

  return debounced;
}

// x Throttle — استاندارد با leading/trailing
function throttle(fn, limit, options = { leading: true, trailing: true }) {
  let inThrottle = false;
  let lastArgs = null;
  let timerId = null;

  const throttled = function (...args) {
    const context = this;

    if (inThrottle) {
      lastArgs = { args, context };
      return;
    }

    if (options.leading) {
      fn.apply(context, args);
    }

    inThrottle = true;
    timerId = setTimeout(() => {
      timerId = null;
      inThrottle = false;
      if (options.trailing && lastArgs) {
        fn.apply(lastArgs.context, lastArgs.args);
        lastArgs = null;
      }
    }, limit);
  };

  throttled.cancel = () => {
    clearTimeout(timerId);
    timerId = null;
    inThrottle = false;
    lastArgs = null;
  };

  return throttled;
}
```

<div dir="rtl">

‏تفاوت کلیدی: Debounce برای «اتمام رویداد» (search as you type, auto-save) و Throttle برای «نرخ ثابت» (scroll, resize, mousemove). هر دو از Closure برای نگهداشتن state (timerId, lastArgs) بدون آلوده کردن scope بیرونی استفاده می‌کنند.

</div>

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏| سناریو | راهکار | دلیل | Trade-off |
‏|--------|--------|------|-----------|
‏| جستجوی لحظه‌ای (search input) | Debounce (250-400ms) | نیازی به اجرا در هر keypress نیست | تأخیر در نمایش نتایج |
‏| Auto-save در editor | Debounce (1000-2000ms) | می‌خواهیم فقط وقتی تایپ متوقف شد ذخیره کنیم | ریسک از دست رفتن داده در خروج ناگهانی |
‏| Scroll event (infinite scroll) | Throttle (100-200ms) | نرخ ثابت لود، بدون درخواست اضافی | ممکن است یک اسکرول سریع را از دست بدهد |
‏| Resize (responsive charts) | Throttle (100-200ms) | محاسبات سنگین نباید در هر پیکسل اجرا شوند | تأخیر در تطبیق UI |
‏| Click (double-submit) | Debounce leading=true | بلافاصله اولین کلیک را اجرا کن، بعدی را نادیده بگیر | کاربر نمی‌تواند سریع دو بار کلیک کند |
‏| Game/animation (keyboard) | Throttle (16-33ms) | هماهنگ با frame rate (60fps ≈ 16ms) | افت precision در ورودی |

</div>

<div dir="rtl">

‏نکته Senior: در React با controlled components، debounce روی **خود setState** کار نمی‌کند — باید روی **source of truth** (مثلاً event handler یا value ورودی) اعمال شود. استفاده از `useMemo` با debounce درون کامپوننت باعث re-creation در هر رندر می‌شود. راه‌حل: `useRef` برای نگهداشتن instance debounce.

</div>

```javascript
// x React — debounce درون hook با useRef
function useDebounce(fn, delay, options) {
  const fnRef = useRef(fn);
  fnRef.current = fn;

  const debouncedRef = useRef();
  if (!debouncedRef.current) {
    debouncedRef.current = debounce((...args) => {
      fnRef.current(...args);
    }, delay, options);
  }

  useEffect(() => {
    const d = debouncedRef.current;
    return () => d.cancel();
  }, []);

  return debouncedRef.current;
}
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏**سناریو:** یک اپلیکیشن dashboard با WebSocket实时 قیمت سهام — هر ثانیه ۶۰-۱۲۰ update می‌آید. هر update باید وضعیت ۵۰۰+ symbol را در Redux به‌روز کند و chart components را re-render کند.

‏**مشکل:** اگر throttle ساده بزنی و آخرین state را در Redux دیسپچ کنی، بین throttle intervals همه‌ی intermediate updates از دست می‌روند. اما اگر همه را دیسپچ کنی، UI لاگ می‌کند.

‏**راه‌حل Production:** «Accumulating Throttle» — در بازه throttle، همه‌ی updates را جمع کن (batch) و یکبار Diff+Patch روی state اعمال کن.

</div>

```javascript
// x Accumulating Throttle — batch تمام updates در یک بازه
function createAccumulatingThrottle(limit) {
  let buffer = [];
  let timerId = null;

  return {
    push(update, applyBatch) {
      buffer.push(update);
      if (!timerId) {
        timerId = setTimeout(() => {
          timerId = null;
          const batch = buffer;
          buffer = [];
          // x یکبار state به‌روز می‌شود
          applyBatch(batch);
        }, limit);
      }
    },
    flush() {
      if (!timerId) return;
      clearTimeout(timerId);
      timerId = null;
      if (buffer.length) {
        const batch = buffer;
        buffer = [];
        return batch;
      }
    },
  };
}

// x استفاده در WebSocket handler
const wsThrottle = createAccumulatingThrottle(100);

ws.onmessage = (event) => {
  const update = JSON.parse(event.data);
  wsThrottle.push(update, (batch) => {
    store.dispatch(batchUpdatePrices(batch));
    // React batched update — React 18 auto-batch
  });
};
```

<div dir="rtl">

‏**نکته Performance:** در Production از `requestAnimationFrame` به جای `setTimeout` برای throttle مرتبط با rendering (scroll, resize) استفاده کن — با VSync هماهنگ است و jank را کاهش می‌دهد.

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏بدون debounce/throttle دستی (مثلاً در محیطی که نمی‌توانی Closure بنویسی یا به ES5 محدودی)، گزینه‌ها:

</div>

| راهکار | موارد استفاده | محدودیت |
|--------|--------------|---------|
| **RxJS** (`.pipe(debounceTime(300))`) | پیچیده، فریم‌ورک غیرمتعلق | وابستگی سنگین، learning curve |
| **Lodash** (`_.debounce` / `_.throttle`) | ساده، battle-tested | ۴KB min+gzip, غیرضروری اگر فقط debounce لازم داری |
| **CSS pointer-events** | جلوگیری از double-click (نه debounce کلی) | فقط UI layer، منطق business را نمی‌گیرد |
| **AbortController** | لغو درخواست قبلی (جایگزین debounce برای fetch) | فقط HTTP requests، توابع عمومی را پوشش نمی‌دهد |

<div dir="rtl">

‏**مقایسه Next.js vs Express.js:** در Next.js با Server Actions، debounce در سمت کلاینت برای auto-save مهم است — در Express.js مجبوری middleware level throttle بزنی. Next.js راه‌حل clean‌تری دارد چون boundary بین client/server مشخص است.

</div>

<div dir="rtl">

‏> **نهایی:** Debounce و Throttle ابزارهای جدایی‌ناپذیر کنترل فرکانس در Frontend هستند. مهم‌ترین نکته Senior: نسخه‌ی ساده (بدون Lodash) را بلد باش، اما در Production از Lodash استفاده نکن — خودت با Closure پیاده‌سازی کن تا بهینه‌سازی خاص پروژه (accumulating, RAF-based) را اضافه کنی. ریسک اصلی: استفاده از debounce در React بدون `useRef` که باعث re-creation در هر render و نشت memory می‌شود.

</div>
