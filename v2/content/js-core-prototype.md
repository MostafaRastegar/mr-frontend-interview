<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Prototype و Prototypal Inheritance (تفاوت آن با Classical Inheritance در جاوا)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Prototype** مکانیزمی در JavaScript است که به اشیاء اجازه می‌دهد ویژگی‌ها و متدها را از یک شیء دیگر به ارث ببرند. هر شیء یک **[[Prototype]]** داخلی دارد (که با `__proto__` یا `Object.getPrototypeOf()` قابل دسترسی است). وقتی یک ویژگی روی شیء پیدا نشود، JavaScript در زنجیره prototype جستجو می‌کند.

</div>

```javascript
// x زنجیره prototype
const parent = { a: 1 };
const child = Object.create(parent);
child.b = 2;

console.log(child.b); // 2 (روی خود child)
console.log(child.a); // 1 (از parent از طریق prototype chain)
console.log(child.toString); // از Object.prototype

// prototype chain: child → parent → Object.prototype → null
console.log(Object.getPrototypeOf(child) === parent);           // true
console.log(Object.getPrototypeOf(parent) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype));            // null
```

<div dir="rtl">

‏### تفاوت با Classical Inheritance (Java)

</div>

<div dir="ltr">

| **ویژگی** | **Prototypal (JS)** | **Classical (Java)** |
|-----------|---------------------|----------------------|
| **مکانیزم** | شیء به شیء (delegation) | کلاس به کلاس (copy) |
| **زمان binding** | runtime (dynamic) | compile-time (static) |
| **تغییر prototype** | ✅ در runtime قابل تغییر | ❌ immutable |
| **this** | dynamic (call-time) | static (compile-time) |
| **Multiple inheritance** | ❌ (زنجیره خطی) | ❌ (در Java single) |
| **Mixin** | ✅ با Object.assign | ❌ (interface default methods) |
| **instanceof** | ✅ prototype chain traversal | ✅ class hierarchy |

</div>

```javascript
// x Prototypal Inheritance در عمل
const animal = {
  speak() { return `${this.name} makes a sound`; }
};

const dog = Object.create(animal);
dog.name = "Rex";
dog.bark = function() { return "Woof!"; };

console.log(dog.speak()); // "Rex makes a sound" — از prototype
console.log(dog.bark());  // "Woof!" — روی خود dog

// x Classical Inheritance (Java-like) در JS با class syntax
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}
class Dog extends Animal {
  bark() { return "Woof!"; }
}
const rex = new Dog("Rex");
console.log(rex.speak()); // "Rex makes a sound"
console.log(rex.bark());  // "Woof!"
```

<div dir="rtl">

‏**تفاوت کلیدی:** در prototypal inheritance، `dog` یک شیء است که به `animal` delegate می‌کند. در classical inheritance، `rex` یک instance از `Dog` است که از `Animal` ارث برده. در JS، `class` فقط syntactic sugar روی prototype است.

</div>

مبحث هفتم تحویل شد. ادامه می‌دم به مبحث هشتم: **Array متدهای پیشرفته (map, filter, reduce, some, every, flatMap)**.

<write_to_file>
<path>v2/content/js-core-array-methods.md</path>
<content><div dir="rtl">

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

</div>

```javascript
// x flatMap: filter + map در یک پاس
const data = [1, 2, 3, 4, 5];
const result = data.flatMap(x => x % 2 === 0 ? [x * 10] : []);
// [20, 40] — filter odd + map even * 10

// x reduce: پیاده‌سازی pipe
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

// reduce: groupBy
const groupBy = (arr, keyFn) =>
  arr.reduce((acc, item) => {
    const key = keyFn(item);
    (acc[key] ||= []).push(item);
    return acc;
  }, {});

const users = [
  { name: "Ali", role: "admin" },
  { name: "Reza", role: "user" },
  { name: "Sara", role: "admin" },
];
console.log(groupBy(users, u => u.role));
// { admin: [{name:"Ali"}, {name:"Sara"}], user: [{name:"Reza"}] }
```

<div dir="rtl">

‏### نکات V8 Optimization

‏- **`map`**: برای آرایه‌های `PACKED_SMI_ELEMENTS` (اعداد صحیح)، V8 از حلقه C++ داخلی استفاده می‌کند — بسیار سریع. اگر callback یک تابع ساده باشد (بدون closure)، V8 می‌تواند آن را inline کند.
‏- **`reduce`**: اگر callback پیچیده باشد (مثلاً با object spread)، V8 نمی‌تواند آن را optimize کند. برای hot paths از حلقه `for` دستی استفاده کنید.
‏- **`flatMap`**: در V8 به صورت یک پاس (single pass) پیاده‌سازی شده — ۲-۳ برابر سریع‌تر از `map().flat()`.

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
```

<div dir="rtl">

‏### نکات V8 Optimization

‏- **`map`**: برای آرایه‌های `PACKED_SMI_ELEMENTS` (اعداد صحیح)، V8 از حلقه C++ داخلی استفاده می‌کند. اگر callback یک تابع ساده باشد (بدون closure)، V8 می‌تواند آن را inline کند.
‏- **`reduce`**: اگر callback پیچیده باشد (مثلاً با object spread)، V8 نمی‌تواند آن را optimize کند. برای hot paths از حلقه `for` دستی استفاده کنید.
‏- **`flatMap`**: در V8 به صورت یک پاس (single pass) پیاده‌سازی شده — ۲-۳ برابر سریع‌تر از `map().flat()`.
‏- **`some`/`every`**: short-circuit — به محض یافتن نتیجه، حلقه متوقف می‌شود. برای آرایه‌های بزرگ حیاتی است.

</div>

```javascript
// Performance comparison
const big = Array.from({ length: 1e6 }, (_, i) => i);

// x ❌ slow: reduce برای transform ساده
const r1 = big.reduce((acc, x) => { acc.push(x * 2); return acc; }, []);

// ✅ fast: map
const r2 = big.map(x => x * 2);

// ❌ slow: filter + map
const r3 = big.filter(x => x % 2 === 0).map(x => x * 10);

// ✅ fast: flatMap
const r4 = big.flatMap(x => x % 2 === 0 ? [x * 10] : []);
```

<div dir="rtl">

‏### نکات V8 Optimization

‏- **`map`**: برای آرایه‌های `PACKED_SMI_ELEMENTS` (اعداد صحیح)، V8 از حلقه C++ داخلی استفاده می‌کند. اگر callback یک تابع ساده باشد (بدون closure)، V8 می‌تواند آن را inline کند.
‏- **`reduce`**: اگر callback پیچیده باشد (مثلاً با object spread)، V8 نمی‌تواند آن را optimize کند. برای hot paths از حلقه `for` دستی استفاده کنید.
‏- **`flatMap`**: در V8 به صورت یک پاس (single pass) پیاده‌سازی شده — ۲-۳ برابر سریع‌تر از `map().flat()`.
‏- **`some`/`every`**: short-circuit — به محض یافتن نتیجه، حلقه متوقف می‌شود. برای آرایه‌های بزرگ حیاتی است.

</div>

```javascript
// Performance comparison
const big = Array.from({ length: 1e6 }, (_, i) => i);

// x ❌ slow: reduce برای transform ساده
const r1 = big.reduce((acc, x) => { acc.push(x * 2); return acc; }, []);

// ✅ fast: map
const r2 = big.map(x => x * 2);

// ❌ slow: filter + map
const r3 = big.filter(x => x % 2 === 0).map(x => x * 10);

// ✅ fast: flatMap
const r4 = big.flatMap(x => x % 2 === 0 ? [x * 10] : []);
```

<div dir="rtl">

‏> **پس از تحویل این مبحث، ادامه بده.**

</div>