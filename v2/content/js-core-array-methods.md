<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Array متدهای پیشرفته (map, filter, reduce, some, every, flatMap)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏شش متد کلیدی روی `Array.prototype` که ستون فقرات برنامه‌نویسی تابعی (Functional Programming) در JavaScript هستند. همه این متدها **غیر مخرب (immutable)** هستند — آرایه اصلی را تغییر نمی‌دهند و یک آرایه/مقدار جدید برمی‌گردانند.

</div>

```javascript
const arr = [1, 2, 3, 4, 5];

// x map: transform هر عنصر → آرایه جدید با همان طول
const doubled = arr.map(x => x * 2); // [2, 4, 6, 8, 10]

// x filter: نگه‌داشتن عناصری که شرط را پاس کنند
const evens = arr.filter(x => x % 2 === 0); // [2, 4]

// x reduce: تجمیع به یک مقدار
const sum = arr.reduce((acc, x) => acc + x, 0); // 15

// x some: آیا حداقل یک عنصر شرط را پاس می‌کند؟
const hasEven = arr.some(x => x % 2 === 0); // true

// x every: آیا همه عناصر شرط را پاس می‌کنند؟
const allPositive = arr.every(x => x > 0); // true

// x flatMap: map + flat(1) در یک پاس
const nested = arr.flatMap(x => [x, x * 2]); // [1,2,2,4,3,6,4,8,5,10]
```

<div dir="rtl">

‏### تفاوت‌های فنی در سطح V8

‏- **`map`**: یک آرایه جدید با طول مساوی می‌سازد. callback برای هر عنصر (شامل `empty` slots) صدا زده می‌شود — برخلاف `forEach` که empty slots را نادیده می‌گیرد.
‏- **`filter`**: یک آرایه جدید با طول ≤ اصلی. اگر callback برای همه عناصر false برگرداند، آرایه خالی برمی‌گردد.
‏- **`reduce`**: یک مقدار واحد. اگر initialValue ندهید و آرایه خالی باشد → `TypeError`. اگر initialValue ندهید و آرایه یک عنصر داشته باشد، آن عنصر برگردانده می‌شود (بدون اجرای callback).
‏- **`some`**: به محض true شدن callback متوقف می‌شود (short-circuit). برای آرایه خالی → `false`.
‏- **`every`**: به محض false شدن callback متوقف می‌شود (short-circuit). برای آرایه خالی → `true` (vacuously true).
‏- **`flatMap`**: معادل `arr.map(fn).flat(1)` اما در یک پاس — بهینه‌تر.

</div>

```javascript
// x reduce: پیاده‌سازی pipe/flow
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

const add1 = x => x + 1;
const double = x => x * 2;
const add1ThenDouble = pipe(add1, double);
console.log(add1ThenDouble(3)); // 8

// x flatMap: filter + map در یک پاس
const words = ["hello world", "foo bar"];
const allWords = words.flatMap(s => s.split(" "));
// ["hello", "world", "foo", "bar"]

// some/every: short-circuit optimization
const bigArray = Array.from({ length: 1e6 }, (_, i) => i);
bigArray.some(x => x === 500); // بعد از ۵۰۱ iteration متوقف می‌شود
bigArray.every(x => x >= 0);   // بعد از ۱ iteration متوقف می‌شود
```

<div dir="rtl">

‏### نکات V8 Optimization

‏- **`map`**: V8 برای آرایه‌های dense (بدون hole) از `ElementsKind: PACKED_SMI_ELEMENTS` استفاده می‌کند و حلقه را inline می‌کند. برای آرایه‌های sparse (با hole) به `HOLEY_*` downgrade می‌شود و کندتر است.
‏- **`reduce`**: اگر initialValue داده نشود، V8 از عنصر اول به عنوان accumulator استفاده می‌کند و از index 1 شروع می‌کند. اگر آرایه خالی باشد → TypeError.
‏- **`flatMap`**: در V8 به صورت یک پاس (single pass) پیاده‌سازی شده — برخلاف `map().flat()` که دو پاس است.
‏- **`some`/`every`**: short-circuit — به محض یافتن نتیجه، حلقه متوقف می‌شود. برای آرایه‌های بزرگ حیاتی است.

</div>

```javascript
// Performance comparison
const big = Array.from({ length: 1e6 }, (_, i) => i);

// ❌ slow: map + flat
const r1 = big.map(x => [x, x * 2]).flat();

// ✅ fast: flatMap
const r2 = big.flatMap(x => [x, x * 2]);

// x ❌ slow: reduce برای transform ساده
const r3 = big.reduce((acc, x) => { acc.push(x * 2); return acc; }, []);

// ✅ fast: map
const r4 = big.map(x => x * 2);

// ❌ slow: filter + map
const r5 = big.filter(x => x % 2 === 0).map(x => x * 10);

// ✅ fast: flatMap
const r6 = big.flatMap(x => x % 2 === 0 ? [x * 10] : []);
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **map** | **filter** | **reduce** | **flatMap** | **some/every** |
|-----------|---------|------------|------------|-------------|---------------|
| **Transform 1:1** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Filter elements** | ❌ | ✅ | ❌ | ❌ | ❌ |
| **Filter + Transform** | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Group/Count/Sum** | ❌ | ❌ | ✅ | ❌ | ❌ |
| **Check existence** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Flatten nested** | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Performance** | ⚡ سریع | ⚡ سریع | ⚠️ کندتر | ⚡ سریع | ⚡ short-circuit |
| **Readability** | ✅ | ✅ | ⚠️ (complex) | ✅ | ✅ |

</div>

<div dir="rtl">

‏### قانون Senior

‏۱. **map**: وقتی ۱ به ۱ transform داری — فقط transform، بدون side effect
‏۲. **filter**: وقتی می‌خواهی عناصری را حذف کنی
‏۳. **reduce**: فقط وقتی map/filter/flatMap کافی نیست — مثل groupBy, zip, compose
‏۴. **flatMap**: وقتی map داری + flatten — همیشه از flatMap استفاده کن
‏۵. **some/every**: برای بررسی شرط روی کل آرایه — از filter().length استفاده نکن

</div>

```javascript
// x ❌ anti-pattern: filter().length برای بررسی وجود
if (arr.filter(x => x > 10).length > 0) { ... } // O(n) کامل

// ✅ some: short-circuit
if (arr.some(x => x > 10)) { ... } // تا پیدا شدن متوقف می‌شود

// x ❌ anti-pattern: forEach با if
let result = [];
arr.forEach(x => { if (x > 0) result.push(x * 2); });

// ✅ flatMap
const result = arr.flatMap(x => x > 0 ? [x * 2] : []);
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: Performance Hit با reduce در پردازش دیتاست بزرگ

‏در یک پروژه داشبورد real-time با ۵۰٬۰۰۰ رکورد، تیم از reduce برای groupBy استفاده کرده بود که باعث jank در UI می‌شد:

</div>

```javascript
// x ***** SLOW: reduce با spread operator *****
const groupByCategory = (items) =>
  items.reduce((acc, item) => ({
    ...acc,
    [item.category]: [...(acc[item.category] || []), item]
  }), {});
// x مشکل: spread در هر iteration → O(n²) — spread یک شیء جدید می‌سازد
```

<div dir="rtl">

‏**مشکل:** ۵۰٬۰۰۰ iteration, هر iteration یک شیء جدید با spread → ۵۰٬۰۰۰ object allocation. GC تحت فشار. UI lag ۲۰۰ms.

</div>

```javascript
// x ✅ FAST: reduce با mutation مستقیم accumulator
const groupByCategory = (items) =>
  items.reduce((acc, item) => {
    (acc[item.category] ??= []).push(item); // mutation عمدی — اینجا مجاز است
    return acc;
  }, {});
// x O(n), 0 object allocation اضافی
```

<div dir="rtl">

‏**نکته Senior:** در reduce، accumulator **مقصد** است — mutation آن acceptable است. قانون مهم: callback باید pure باشد، نه accumulator. accumulator صرفاً یک container است.

</div>

```javascript
// x چالش دیگر: پردازش داده‌های nested API
const apiData = [
  { id: 1, tags: ["a", "b"] },
  { id: 2, tags: ["b", "c"] },
  { id: 3, tags: null },       // null tag
  { id: 4 },                   // بدون tags
];

// Junior approach
const allTags = apiData.flatMap(item => item.tags || []).filter(Boolean);

// x Senior approach — با type safety و edge case handling
const allTags = apiData
  .filter((item): item is { id: number; tags: string[] } => 
    Array.isArray(item.tags)
  )
  .flatMap(item => item.tags);

// x همچنین: deduplication با reduce
const uniqueTags = [...new Set(allTags)];
```

<div dir="rtl">

‏### Nested reduce برای Multi-level GroupBy

</div>

```javascript
// x داده: تراکنش‌های فروش
const transactions = [
  { year: 2024, month: "Jan", amount: 100 },
  { year: 2024, month: "Jan", amount: 200 },
  { year: 2024, month: "Feb", amount: 150 },
  { year: 2025, month: "Jan", amount: 300 },
];

// GroupBy year → month → sum
const grouped = transactions.reduce((acc, t) => {
  acc[t.year] ??= {};
  acc[t.year][t.month] ??= 0;
  acc[t.year][t.month] += t.amount;
  return acc;
}, {});
// { 2024: { Jan: 300, Feb: 150 }, 2025: { Jan: 300 } }
```

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزینه</div>

<div dir="rtl">

‏اگر JavaScript نبود و در یک زبان بدون متدهای functional array (مثل C یا Go قدیم) بودم:

</div>

<div dir="ltr">

| **نیاز** | **JS (Array methods)** | **Go (بدون generics)** | **Rust** |
|----------|----------------------|------------------------|----------|
| Transform | `map` | `for` loop | `.iter().map()` |
| Filter | `filter` | `for` loop + append | `.iter().filter()` |
| Accumulate | `reduce` | `for` loop | `.iter().fold()` |
| FlatMap | `flatMap` | nested loop | `.iter().flat_map()` |
| Any/All | `some/every` | `for` + break | `.iter().any()/.all()` |

</div>

<div dir="rtl">

‏**برنده:** JavaScript در readability و conciseness برنده است. در performance بحرانی، حلقه `for` دستی در JS همیشه از `map/filter/reduce` سریع‌تر است (چون overhead تابع callback را ندارد).

</div>

```javascript
// x Hot path: حلقه for دستی ۲-۳ برابر سریع‌تر از map
// for
let result = new Array(arr.length);
for (let i = 0; i < arr.length; i++) {
  result[i] = arr[i] * 2;
}

// vs map
const result = arr.map(x => x * 2);

// x دلیل: map overhead دارد: callback creation, function call, bound checking
// x ولی در ۹۹٪ موارد تفاوت قابل توجه نیست — readability را priority بده
```

<div dir="rtl">

‏> **نهایی:** متدهای Array ستون فقرات FP در JS هستند. قانون: readability اول، performance بعد. برای hot paths (۱۰٬۰۰۰+ عنصر در حلقه‌های بحرانی) از `for` استفاده کن. برای بقیه جاها از متدهای Array.

</div>