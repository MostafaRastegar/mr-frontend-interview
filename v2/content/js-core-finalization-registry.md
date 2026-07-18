<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "FinalizationRegistry برای مدیریت حافظه و پاک‌سازی منابع" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**FinalizationRegistry** یک API است که به شما اجازه می‌دهد callbackای ثبت کنید که پس از **GC شدن** یک object (زمانی که هیچ reference قوی به آن وجود ندارد) اجرا می‌شود.

‏**WeakRef** — یک reference ضعیف به یک object که اگر GC هنوز آن را پاک نکرده باشد، می‌توانی از آن استفاده کنی؛ در غیر این صورت `undefined` برمی‌گرداند.

</div>

```javascript
// x FinalizationRegistry — clean-up پس از GC
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`Object ${heldValue} was garbage collected`);
  // x پاک‌سازی: بستن connection, آزاد کردن handle, گزارش
});

let obj = { data: 'heavy' };
registry.register(obj, 'my-obj-id');
// x یا با unregister token:
const token = {};
registry.register(obj, 'my-obj-id', token);
// x registry.unregister(token); // لغو

obj = null; // GC eligible → eventually callback runs

// x WeakRef — دسترسی موقت
let heavy = new ArrayBuffer(1024 * 1024 * 100); // 100MB
const ref = new WeakRef(heavy);

function getHeavy() {
  const cached = ref.deref();
  if (cached !== undefined) {
    return cached; // هنوز زنده است
  }
  // x GC شد — دوباره بساز
  heavy = new ArrayBuffer(1024 * 1024 * 100);
  ref = new WeakRef(heavy);
  return heavy;
}
```

<div dir="rtl">

‏**محدودیت مهلک:** `finalization` و `deref` **غیرقابل پیش‌بینی** هستند — نمی‌توانی تضمین کنی کی اجرا می‌شوند یا اصلاً اجرا می‌شوند. هرگز برای منطق business-critical استفاده نکن (مثل ذخیره‌سازی داده).

</div>

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏| استفاده | مناسب | نامناسب |
‏|---------|-------|---------|
‏| آزادسازی FileHandle/WebSocket | ✅ GC شده → بستن | ❌ کلید خارجی در Cache |
‏| پاک‌سازی DOM observer (ResizeObserver) | ✅ بدون نشت memory | ❌ تراکنش مالی |
‏| Reporting/metrics | ✅ اطلاع از GC pattern | ❌ تضمین پاک‌سازی resource |

</div>

```javascript
// x Cache با WeakRef — خودپاک‌شونده
class WeakCache {
  constructor() {
    this.#registry = new FinalizationRegistry((key) => {
      this.#index.delete(key);
    });
    this.#index = new Map();
  }

  get(key, factory) {
    const entry = this.#index.get(key);
    if (entry) {
      const val = entry.ref.deref();
      if (val !== undefined) return val;
      // x GC شد — cleanup index
      this.#index.delete(key);
    }
    const value = factory();
    const ref = new WeakRef(value);
    this.#index.set(key, { ref });
    this.#registry.register(value, key, ref);
    return value;
  }
}
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏**سناریو:** یک SPA با ۵۰+ WebSocket connection (هر کدام برای symbol بازار سهام). کاربر بین صفحه‌ها جابه‌جا می‌شود — کامپوننت‌ها unmount می‌شوند اما بعضی WebSocket‌ها به دلیل closure مرجع بسته نمی‌شوند.

‏**چالش:** بدون FinalizationRegistry، هر بار mount شدن یک symbol، connection جدید باز می‌شود — connections قبلی به مرور انباشته می‌شوند (nشت حافظه).

‏**راه‌حل:** FinalizationRegistry به عنوان safety net برای بستن connections.

</div>

```javascript
class StockConnection {
  constructor(symbol, onPrice) {
    this.symbol = symbol;
    this.ws = new WebSocket(`wss://prices/${symbol}`);
    this.ws.onmessage = (e) => onPrice(JSON.parse(e.data));

    // x Safety net — اگر کامپوننت بدون clean-up نابود شد
    this.#cleanup = new FinalizationRegistry((held) => {
      const { ws, symbol } = held;
      if (ws.readyState === WebSocket.OPEN) {
        console.warn(`[GC] Closing leaked connection for ${symbol}`);
        ws.close();
      }
    });
    this.#cleanup.register(this, { ws: this.ws, symbol }, this);
  }

  destroy() {
    this.#cleanup.unregister(this);
    this.ws.close();
  }
}

// x React hook با safety net
function useStockPrice(symbol) {
  const connRef = useRef(null);

  useEffect(() => {
    const conn = new StockConnection(symbol, (price) => {
      setPrice(price);
    });
    connRef.current = conn;

    return () => {
      conn.destroy(); // proper clean-up
      connRef.current = null;
    };
  }, [symbol]);

  return price;
}
```

<div dir="rtl">

‏**نکته Production:** FinalizationRegistry را **جایگزین** `useEffect` cleanup نکن — همیشه `destroy()` را در `useEffect` برگردان. FR فقط safety net است برای edge cases (مثل خطاهای حافظه‌ای, hot reload, race conditions در StrictMode).

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏بدون FinalizationRegistry (پشتیبانی limited):

</div>

| راهکار | روش | محدودیت |
|--------|-----|---------|
| `dispose` pattern | کلاس با `destroy()` explicit | وابستگی به caller |
| AbortController | لغو in-flight requests | فقط async operations |
| `useEffect` cleanup | React lifecycle | فقط در React, نه JS عمومی |
| `addEventListener` options | `{ once: true }` | فقط event listeners |

<div dir="rtl">

‏**مقایسه Next.js vs Express.js:** در Next.js Server Components (سرور)، need به FinalizationRegistry کمتر است — هر request lifecycle کوتاه دارد. در Express.js با long-lived connections (WebSocket در same server), FinalizationRegistry می‌تواند کمک کند. در هر دو پلتفرم, GC-dependent operations غیرقابل اعتماد هستند — از explicit clean-up با `AbortController` یا `dispose` pattern استفاده کن.

</div>

<div dir="rtl">

‏> **نهایی:** FinalizationRegistry یک ابزار niche است. برای ۹۹٪ موارد, explicit clean-up در `useEffect` یا `destroy()` کافی است. از FR فقط برای safety net استفاده کن — هرگز برای منطق اصلی. مهم‌ترین نکته: GC ممکن است **هیچ‌وقت** اجرا نشود (حافظه کافی وجود دارد) — نمی‌توانی روی FR برای آزادسازی resourceهای محدود (file handles, connections) حساب کنی.

</div>
