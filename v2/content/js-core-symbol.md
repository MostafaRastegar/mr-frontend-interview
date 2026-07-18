<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Symbol و کاربرد آن در ایجاد کلیدهای خصوصی در آبجکت‌ها و جلوگیری از تداخل نام‌ها" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Symbol** یک نوع داده primitive در ES6 است که یک مقدار unique و immutable را نمایندگی می‌کند. هر فراخوانی `Symbol()` یک شناسه جدید و منحصربه‌فرد می‌سازد که با هیچ Symbol دیگری برابر نیست، حتی اگر description یکسان داشته باشند. کاربرد اصلی: کلیدهای خصوصی در آبجکت‌ها و جلوگیری از تداخل نام‌ها.

</div>

```javascript
// x ایجاد Symbol
const sym1 = Symbol("description");
const sym2 = Symbol("description");

console.log(sym1 === sym2); // false — همیشه unique
console.log(sym1.toString()); // "Symbol(description)"

// x Symbol به عنوان کلید در آبجکت
const obj = {};
const id = Symbol("id");
obj[id] = "secret";
console.log(obj[id]); // "secret"
console.log(Object.keys(obj)); // [] — Symbol key لو نمی‌رود
```

<div dir="rtl">

‏### خصوصیات فنی در سطح V8/موتور

‏- **Unique identity**: هر Symbol یک internal slot `[[SymbolData]]` دارد که identity آن را در موتور مشخص می‌کند.
‏- **Not auto-converted to string**: برخلاف其他 primitive‌ها، Symbol نمی‌تواند به string تبدیل شود — `String(sym)` مجاز است ولی `sym + ""` خطا می‌دهد.
‏- **Well-known Symbols**: مجموعه‌ای از Symbolهای داخلی مثل `Symbol.iterator`، `Symbol.toStringTag`، `Symbol.hasInstance` که رفتار داخلی اشیاء را سفارشی می‌کنند.
‏- **Global Symbol Registry**: `Symbol.for(key)` یک Symbol مشترک بین realm‌ها (مثل iframe) ایجاد می‌کند.

</div>

```javascript
// x Well-known Symbols — سفارشی‌سازی رفتار داخلی
class MyArray {
  [Symbol.iterator]() {
    let i = 0;
    return {
      next: () => ({ value: i++, done: i > 5 })
    };
  }
}
const arr = new MyArray();
console.log([...arr]); // [0, 1, 2, 3, 4]

// Symbol.toStringTag
class MyClass {
  get [Symbol.toStringTag]() { return "MyClass"; }
}
console.log(Object.prototype.toString.call(new MyClass())); // "[object MyClass]"

// Global Symbol Registry
const a = Symbol.for("app.key");
const b = Symbol.for("app.key");
console.log(a === b); // true — در سراسر realm مشترک
```

<div dir="rtl">

‏### Symbol vs String key

</div>

```javascript
const obj = {};

// x string key — در معرض تداخل
obj.name = "ali";

// x Symbol key — ایزوله
const nameSym = Symbol("name");
obj[nameSym] = "secret";

console.log(obj.name);        // "ali" — string key
console.log(obj[nameSym]);    // "secret" — Symbol key

// x Symbol key در Object.keys و for..in دیده نمی‌شود
console.log(Object.keys(obj)); // ["name"]
console.log(Object.getOwnPropertyNames(obj)); // ["name"]
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(name)]
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **Symbol** | **String key** | **WeakMap** |
|-----------|-----------|-----------------|-------------|
| **مقاوم در برابر تداخل نام** | ✅ | ❌ | ✅ |
| **قابل دسترسی در Object.keys()** | ❌ | ✅ | ❌ |
| **قابل دسترسی در JSON.stringify()** | ❌ | ✅ | ❌ |
| **قابل دسترسی در for..in** | ❌ | ✅ | ❌ |
| **قابل دسترسی در spread** | ❌ | ✅ | ❌ |
| **قابلیت سریالایز شدن** | ❌ | ✅ | ❌ |
| **Memory management** | خودکار | خودکار | ✅ (WeakRef) |
| **Performance** | ⚡ هم‌سطح string | ⚡ | ⚠️ کمی کندتر |

</div>

<div dir="rtl">

‏### قانون Senior

‏۱. **Symbol**: وقتی نیاز به کلید خصوصی داری که نباید با کتابخانه‌های third-party تداخل کند.
‏۲. **String key**: برای داده‌هایی که باید JSON.stringify شوند یا در API ظاهر شوند.
‏۳. **WeakMap**: وقتی عمر کلید به طول عمر شیء گره خورده است (جلوگیری از memory leak).

</div>

```javascript
// x Symbol برای metadata داخلی کتابخانه
const METADATA = Symbol("metadata");

class User {
  constructor(data) {
    this[METADATA] = { createdAt: Date.now(), source: "api" };
    Object.assign(this, data);
  }
}

const user = new User({ name: "Ali" });
console.log(user.name);       // "Ali"
console.log(user[METADATA]);  // { createdAt: ..., source: "api" }
// x metadata در API response ظاهر نمی‌شود
console.log(JSON.stringify(user)); // {"name":"Ali"}
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: تداخل کلید در آبجکت‌های اشتراکی

‏در یک پروژه Micro-frontend با Module Federation، دو تیم مختلف روی یک آبجکت global کار می‌کردند. تیم A یک خاصیت `status` روی آبجکت response اضافه کرد. تیم B هم از `status` برای مقصود دیگری استفاده کرد → تداخل.

</div>

```javascript
// x مشکل: تداخل نام
// x تیم A
function processResponse(data) {
  data.status = "processed"; // تداخل با تیم B
  return data;
}

// x تیم B
function enrichResponse(data) {
  data.status = "enriched"; // بازنویسی status تیم A
  return data;
}
```

<div dir="rtl">

‏**راه‌حل با Symbol:**

</div>

```javascript
// x تیم A
const STATUS_A = Symbol("status");
function processResponse(data) {
  data[STATUS_A] = "processed";
  return data;
}

// x تیم B
const STATUS_B = Symbol("status");
function enrichResponse(data) {
  data[STATUS_B] = "enriched";
  return data;
}

// x همزیستی مسالمت‌آمیز — بدون تداخل
const data = {};
processResponse(data);
enrichResponse(data);
console.log(data[STATUS_A]); // "processed"
console.log(data[STATUS_B]); // "enriched"
```

<div dir="rtl">

‏### سناریوی واقعی: Well-known Symbol برای سفارشی‌سازی

‏در یک کتابخانه validation، از `Symbol.toPrimitive` برای تبدیل custom type به string استفاده کردیم:

</div>

```javascript
class Email {
  #value;
  constructor(value) {
    if (!value.includes("@")) throw new Error("Invalid email");
    this.#value = value;
  }

  // x سفارشی‌سازی تبدیل به primitive
  [Symbol.toPrimitive](hint) {
    if (hint === "string") return this.#value;
    return this.#value;
  }

  // x representation مناسب برای debug
  get [Symbol.toStringTag]() { return "Email"; }
}

const email = new Email("ali@example.com");
console.log(String(email));         // "ali@example.com"
console.log(email + "");            // "ali@example.com" (hint: default)
console.log(Object.prototype.toString.call(email)); // "[object Email]"
```

<div dir="rtl">

‏### Symbol.replace برای سفارشی‌سازی String.prototype.replace

</div>

```javascript
class Redactor {
  [Symbol.replace](str, replacement) {
    // x redact فقط ایمیل‌ها
    return str.replace(/[\w.-]+@[\w.-]+\.\w+/g, replacement);
  }
}

const redactor = new Redactor();
const text = "contact me at ali@example.com";
console.log(text.replace(redactor, "[REDACTED]"));
// "contact me at [REDACTED]"
```

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزینه</div>

<div dir="rtl">

‏اگر Symbol در دسترس نبود (یا در محیطی که باید آبجکت سریالایز شود):

</div>

<div dir="ltr">

| **روش** | **Symbol** | **WeakMap** | **closure + closure variable** | **underscore convention** |
|---------|-----------|-------------|-------------------------------|---------------------------|
| **خصوصی واقعی** | ✅ | ✅ | ✅ | ❌ (قراردادی) |
| **قابل سریالایز** | ❌ | ❌ | ❌ | ✅ |
| **قابل دسترسی در debug** | ❌ | ❌ | ❌ | ✅ |
| **Memory leak** | ❌ | ✅ (WeakRef) | ❌ | ❌ |
| **Performance** | ⚡ | ⚠️ | ⚡ | ⚡ |

</div>

```javascript
// x جایگزین Symbol با WeakMap برای metadata خصوصی
const metadata = new WeakMap();

class User {
  constructor(data) {
    metadata.set(this, { createdAt: Date.now() });
    Object.assign(this, data);
  }

  get createdAt() {
    return metadata.get(this)?.createdAt;
  }
}

// x جایگزین Symbol با closure
function createPrivateObj() {
  const privateData = new Map();
  const key = {};

  return {
    set secret(val) { privateData.set(key, val); },
    get secret() { return privateData.get(key); }
  };
}

// x جایگزین Symbol با underscore convention (ضعیف)
const obj = {
  _private: "secret", // فقط قراردادی — همچنان قابل دسترسی
  public: "visible"
};
```

<div dir="rtl">

‏> **نهایی:** Symbol بهترین انتخاب برای کلیدهای مقاوم در برابر تداخل است. محدودیت: در JSON.stringify و spread نادیده گرفته می‌شود. اگر نیاز به سریالایز داری، از Map/WeakMap استفاده کن. Well-known Symbols (مثل `Symbol.iterator`) برای سفارشی‌سازی رفتار داخلی اشیاء در کتابخانه‌ها ضروری هستند.

</div>