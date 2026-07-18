<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Deep Copy vs Shallow Copy (مشکل `structuredClone` در مرورگرهای قدیمی و پیاده‌سازی با JSON.stringify)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Shallow Copy:** فقط سطح اول object کپی می‌شود. nested objects/references به همان حافظه اشاره می‌کنند — mutation در clone، original را تغییر می‌دهد.

‏**Deep Copy:** کل ساختار object به صورت recursive کپی می‌شود — هیچ reference مشترکی بین clone و original باقی نمی‌ماند.

</div>

```javascript
// x Shallow Copy — ۳ روش رایج
const a = { x: 1, y: { z: 2 } };
const spread = { ...a };
const assign = Object.assign({}, a);
const shallow = Object.create(a); // prototype chain

spread.y.z = 99;
console.log(a.y.z); // 99 — mutation leaked!

// x Deep Copy — ۳ روش اصلی
const json = JSON.parse(JSON.stringify(a));        // روش ۱
const structured = structuredClone(a);              // روش ۲
const recursive = deepClone(a);                     // روش ۳ (دستی)

// JSON.stringify limitations:
JSON.parse(JSON.stringify({
  undef: undefined,           // dropped
  fn: () => {},               // dropped
  sym: Symbol('x'),           // dropped
  big: 1n,                    // TypeError
  date: new Date(),           // becomes string
  regex: /foo/g,              // becomes {}
  err: new Error('fail'),     // becomes {}
  inf: Infinity,              // becomes null
  nan: NaN,                   // becomes null
  map: new Map([['a', 1]]),   // becomes {}
  set: new Set([1, 2]),       // becomes {}
  circ: {},                   // TypeError: circular
}));
```

<div dir="rtl">

‏`structuredClone` (Web API, 2022+) مشکلات JSON.stringify را حل می‌کند: Date, Map, Set, RegExp, Error, ArrayBuffer, circular references را پشتیبانی می‌کند. محدودیت: `Symbol`, `Function`, `WeakMap`, `WeakSet`, DOM nodes, prototype chain را نمی‌تواند.

</div>

```javascript
// x Deep Clone دستی — full-featured
function deepClone(value, seen = new WeakMap()) {
  if (value === null || typeof value !== 'object') return value;

  // Handle circular references
  if (seen.has(value)) return seen.get(value);

  // Primitive wrappers & Date
  if (value instanceof Date) return new Date(value.getTime());
  if (value instanceof RegExp) return new RegExp(value.source, value.flags);
  if (value instanceof Map) {
    const clone = new Map();
    seen.set(value, clone);
    value.forEach((v, k) => clone.set(deepClone(k, seen), deepClone(v, seen)));
    return clone;
  }
  if (value instanceof Set) {
    const clone = new Set();
    seen.set(value, clone);
    value.forEach(v => clone.add(deepClone(v, seen)));
    return clone;
  }
  if (value instanceof ArrayBuffer) return value.slice(0);
  if (ArrayBuffer.isView(value)) return new (value.constructor)(value.slice());

  // Plain object / class instance
  const proto = Object.getPrototypeOf(value);
  const clone = Object.create(proto || null);
  seen.set(value, clone);
  for (const key of [...Object.keys(value), ...Object.getOwnPropertySymbols(value)]) {
    clone[key] = deepClone(value[key], seen);
  }
  return clone;
}
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏| راهکار | سرعت | پشتیبانی | محدودیت‌ها | استفاده |
‏|--------|------|----------|------------|---------|
‏| Spread/assign | O(1) سطح اول | همه جا | shallow فقط | روزمره، state update سطح اول |
‏| `JSON.parse(JSON.stringify)` | O(n) | همه جا | undefined, function, Symbol, circular, Date, Map... | data-only object (API response) |
‏| `structuredClone` | O(n), native | Chrome 98+, FF 94+, Safari 15.4+ | No Symbol/Function/Weak/Prototype | production — بهترین انتخاب ممکن |
‏| Lodash `_.cloneDeep` | O(n), JS | IE9+ | حجم (۴KB+gzip) | legacy browser, نیاز به full-featured |
‏| deepClone دستی | O(n), سفارشی | customizable | نگهداری, edge case | وقتی typeهای خاص نیاز داری (Map, Buffer) |

</div>

<div dir="rtl">

‏نکته Senior: `structuredClone` در سرویس‌ورکرها و Web Workers built-in است (postMessage از آن استفاده می‌کند) — اگر محیطت مدرن است، هیچ دلیلی برای کتابخانه یا پیاده‌سازی دستی وجود ندارد. محدودیت واقعی: انتقال prototype chain — نمی‌توانی instance یک class سفارشی را clone کنی؛ برای آن باید از custom revive استفاده کنی.

</div>

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏**سناریو:** یک React dashboard با Redux-Toolkit (immer). کاربر یک فرم پیچیده با nested array از آیتم‌ها دارد — هر آیتم شامل Date, File reference, Map از metadata است. می‌خواهی undo/redo با full snapshot پیاده‌کنی.

‏**چالش:** `JSON.parse(JSON.stringify(state))` تاریخ‌ها را string می‌کند، `structuredClone` File/Promise/Blob را می‌اندازد، spread روی sub-array shallow است و undo تاریخچه را خراب می‌کند چون mutations از طریق history propagate می‌شوند.

‏**راه‌حل:** deepClone سفارشی با type-aware revive + نسخه‌ی immer-compatible برای undo stack.

</div>

```javascript
// x Type-aware deep clone برای undo stack
const TYPE_HANDLERS = {
  Date: (v) => new Date(v.getTime()),
  Map: (v) => new Map(JSON.parse(JSON.stringify([...v]))),
  Set: (v) => new Set(JSON.parse(JSON.stringify([...v]))),
  File: (v) => v,  // immutable — می‌توانی reference را نگه داری
};

function safeClone(state, seen = new WeakMap()) {
  if (state === null || typeof state !== 'object') return state;
  if (seen.has(state)) return seen.get(state);

  const handler = TYPE_HANDLERS[state.constructor?.name];
  if (handler) {
    const clone = handler(state);
    seen.set(state, clone);
    return clone;
  }

  if (Array.isArray(state)) {
    const clone = state.map(item => safeClone(item, seen));
    seen.set(state, clone);
    return clone;
  }

  const proto = Object.getPrototypeOf(state);
  const clone = Object.create(proto);
  seen.set(state, clone);
  for (const key of Reflect.ownKeys(state)) {
    const desc = Object.getOwnPropertyDescriptor(state, key);
    if (desc.get || desc.set) {
      Object.defineProperty(clone, key, desc);
    } else {
      clone[key] = safeClone(state[key], seen);
    }
  }
  return clone;
}

// x Redux middleware برای undo history
const undoMiddleware = (maxHistory = 50) => {
  let past = [];
  return (store) => (next) => (action) => {
    if (action.type === 'UNDO') {
      const prev = past.pop();
      if (prev) store.dispatch({ type: 'SET_STATE', payload: prev });
      return;
    }
    if (action.type !== 'SET_STATE') {
      const snapshot = safeClone(store.getState());
      past.push(snapshot);
      if (past.length > maxHistory) past.shift();
    }
    return next(action);
  };
};
```

<div dir="rtl">

‏**نکته Performance:** snapshotهای کامل undo stack حافظه را اشغال می‌کنند. برای stateهای بزرگ، از structural sharing (Immer) یا差分 snapshot (MongoDB style oplog) استفاده کن. در production از `redux-undo` یا `zundo` برای Zustand استفاده کن — نه deepClone کامل.

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏بدون deep clone (مثلاً در محیط محدود ES5 یا IE11)، گزینه‌ها:

</div>

| راهکار | توضیح | محدودیت |
|--------|-------|---------|
| **Immutability helpers** (Immer, Immutable.js) | تغییرات روی draft اعمال می‌شوند، original untouched | وابستگی, migration هزینه دارد |
| **Structural sharing** (redux-toolkit) | immer produce → state قبلی immutable می‌ماند | فقط برای Redux ecosystem |
| **Patch-based** (JSON Patch, RFC 6902) | فقط diff ذخیره می‌شود نه full state | پیچیدگی در apply/rollback |
| **Manual copy** (Object.assign عمیق با حلقه) | وقتی تعداد فیلدها محدود است (DTO) | maintainability پایین |

<div dir="rtl">

‏**مقایسه Next.js vs Express.js:** در Next.js Server Components، داده‌ها بین سرور و کلاینت serialized می‌شوند — `structuredClone` در عمل همان الگوریتم serialization سرور را استفاده می‌کند. اگر تایپ‌های غیرقابل clone (Maps/Sets) از سرور برگردانده شوند، هنگام `use cache` با خطا مواجه می‌شوی. در Express.js این مشکل نیست چون State سمت سرور mutable است و serialization فقط در response boundary رخ می‌دهد.

</div>

<div dir="rtl">

‏> **نهایی:** برای داده‌های plain (API JSON) از `JSON.parse(JSON.stringify)` استفاده کن — ساده و همه‌جا کار می‌کند. برای production مدرن (Chrome 98+, FF 94+) از `structuredClone` استفاده کن — native، سریع، و مشکلات circular/Date/Map را حل می‌کند. عمیقاً نکته Senior: درک این که Spread `{...obj}` و Object.assign shallow هستند، مهم‌ترین چیزی است که از تو می‌خواهند — و اینکه mutation از طریق reference در Redux/Zustand state باعث bugs صعب‌العلاج می‌شود.

</div>
