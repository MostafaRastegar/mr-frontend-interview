<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "WeakMap و WeakSet (تفاوت با Map/Set و کاربرد در مدیریت حافظه و جلوگیری از Memory Leak در Cache)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**WeakMap** و **WeakSet** انواع داده‌ای در ES6 هستند که مشابه Map و Set عمل می‌کنند اما با یک تفاوت بنیادی: کلیدهای آن‌ها **weakly referenced** هستند. یعنی اگر هیچ reference دیگری به یک شیء (کلید) وجود نداشته باشد، GC می‌تواند آن را پاک کند — حتی اگر در WeakMap/WeakSet باشد.

</div>

```javascript
// x WeakMap — کلیدها حتماً Object (نه primitive)
const wm = new WeakMap();
let obj = { id: 1 };
wm.set(obj, "metadata");
console.log(wm.get(obj)); // "metadata"

obj = null; // GC می‌تواند این entry را پاک کند
// x wm.size undefined — WeakMap size را افشا نمی‌کند

// x WeakSet — مشابه
const ws = new WeakSet();
let user = { name: "ali" };
ws.add(user);
console.log(ws.has(user)); // true
user = null; // GC می‌تواند پاک کند
```

<div dir="rtl">

‏### خصوصیات فنی در سطح V8/موتور

‏- **Weak references**: کلیدها weakly referenced هستند — GC می‌تواند آن‌ها را جمع‌آوری کند حتی اگر در WeakMap/WeakSet باشند.
‏- **Non-iterable**: WeakMap و WeakSet iterable نیستند — چون محتوا در هر لحظه می‌تواند تغییر کند (GC).
‏- **No size property**: چون محتوا ناپایدار است، `size` یا `length` ندارند.
‏- **Keys must be objects**: کلیدها حتماً Object (یا Symbol در WeakMap) — primitive مجاز نیست.
‏- **Internal WeakRef**: هر entry یک `WeakRef` به کلید نگه می‌دارد — GC از طریق `gc` callback مطلع می‌شود.

</div>

```javascript
// x WeakMap — فقط کلید Object
const wm = new WeakMap();
// wm.set("key", "val"); // TypeError: Invalid value used as weak map key
let ref = { id: 1 };
wm.set(ref, { data: "secret" });

// WeakSet
const ws = new WeakSet();
ws.add(ref);
console.log(ws.has(ref)); // true

ref = null; // هر دو entry آماده GC
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **Map** | **WeakMap** | **Object** |
|-----------|---------|-------------|------------|
| **کلید Object** | ✅ | ✅ | ✅ (string key) |
| **کلید primitive** | ✅ | ❌ | ✅ |
| **Iterable** | ✅ | ❌ | ❌ (Object.keys) |
| **قابل سریالایز** | ❌ | ❌ | ✅ (JSON) |
| **Memory leak prevention** | ❌ | ✅ | ❌ |
| **Performance** | ⚡ | ⚠️ (GC overhead) | ⚡ |
| **size property** | ✅ | ❌ | ❌ |

</div>

<div dir="rtl">

‏### قانون Senior

‏۱. **WeakMap**: برای cache موقت که با حذف شیء منبع پاک شود — جلوگیری از memory leak.
‏۲. **WeakSet**: برای tracking اشیاء بدون نگه‌داشتن reference قوی (مثلاً علامت‌گذاری اشیاء پردازش‌شده).
‏۳. **Map**: وقتی نیاز به iteration یا size داری.
‏۴. **Object**: وقتی نیاز به سریالایز داری.

</div>

```javascript
// x WeakMap برای cache موقت
const cache = new WeakMap();

function processExpensive(obj) {
  if (cache.has(obj)) {
    console.log("cache hit");
    return cache.get(obj);
  }
  const result = { /* expensive computation */ };
  cache.set(obj, result);
  return result;
}

// x WeakSet برای tracking
const processed = new WeakSet();
function markProcessed(obj) {
  processed.add(obj);
}
function isProcessed(obj) {
  return processed.has(obj);
}
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: Memory Leak در Cache با Map

‏در یک پروژه Dashboard با داده‌های real-time، از Map برای کش کردن metadata اشیاء DOM استفاده کردیم. بعد از حذف DOM elements از صفحه، Map همچنان reference به آن‌ها را نگه داشته بود → memory leak.

</div>

```javascript
// x مشکل: Memory Leak با Map
const cache = new Map();

function trackElement(el) {
  cache.set(el, { timestamp: Date.now(), handler: () => {} });
}

// x بعد از حذف element از DOM
const el = document.querySelector(".widget");
trackElement(el);
el.remove(); // element از DOM رفت
// x اما cache همچنان reference به el دارد → memory leak
```

<div dir="rtl">

‏**راه‌حل با WeakMap:**

</div>

```javascript
// x راه‌حل: WeakMap — GC خودکار
const wmCache = new WeakMap();

function trackElement(el) {
  wmCache.set(el, { timestamp: Date.now(), handler: () => {} });
}

const el = document.querySelector(".widget");
trackElement(el);
el.remove(); // element از DOM رفت
// x GC می‌تواند el را پاک کند → cache هم خودکار پاک می‌شود
// x بدون memory leak
```

<div dir="rtl">

‏### سناریوی واقعی: Cache در کتابخانه React-like

‏در یک کتابخانه state management، از WeakMap برای نگه‌داشتن state هر کامپوننت استفاده کردیم:

</div>

```javascript
// x WeakMap برای state کامپوننت‌ها
const componentState = new WeakMap();

function createComponent(instance) {
  componentState.set(instance, { mounted: true, data: {} });
  return {
    getState: () => componentState.get(instance),
    destroy: () => {
      // x نیازی به پاک کردن WeakMap نیست — GC خودکار
      instance = null;
    }
  };
}
```

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزینه</div>

<div dir="rtl">

‏اگر WeakMap/WeakSet در دسترس نبود:

</div>

<div dir="ltr">

| **روش** | **WeakMap** | **Map + manual cleanup** | **Symbol key** | **FinalizationRegistry** |
|---------|-------------|--------------------------|----------------|--------------------------|
| **GC خودکار** | ✅ | ❌ | ❌ | ✅ |
| **پیچیدگی** | کم | متوسط | کم | زیاد |
| **قابلیت debug** | کم | زیاد | کم | متوسط |
| **کنترل دستی** | ❌ | ✅ | ❌ | ✅ (callback) |

</div>

```javascript
// x جایگزین WeakMap با Map + manual cleanup (ضعیف)
const mapCache = new Map();
const el = document.querySelector(".widget");
mapCache.set(el, { data: "..." });
// x باید حتماً در cleanup:
el.addEventListener("remove", () => mapCache.delete(el));

// x جایگزین با FinalizationRegistry (پیشرفته)
const registry = new FinalizationRegistry((key) => {
  console.log(`Object ${key} collected`);
});
registry.register(el, "el-1");
```

<div dir="rtl">

‏> **نهایی:** WeakMap/WeakSet برای سناریوهایی که عمر داده به عمر شیء گره خورده است (مثل metadata DOM elements، cache موقت، tracking) بهترین انتخاب هستند. محدودیت: non-iterable و non-serializable. اگر نیاز به iteration داری، از Map با manual cleanup استفاده کن.

</div>