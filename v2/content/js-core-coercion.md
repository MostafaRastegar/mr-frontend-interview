<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Coercion (تبدیل ضمنی نوع‌ها) و متدهای `ToPrimitive` و `toStringTag`" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏Coercion تبدیل خودکار یک مقدار از یک نوع به نوع دیگر در زمان اجراست. JS دو نوع coercion دارد:

‏- **Implicit:** توسط engine در عملیات `+`, `==`, `if()`, `!` و ...
‏- **Explicit:** با `Number()`, `String()`, `Boolean()`, `parseInt`, `toString`

‏الگوریتم `ToPrimitive`: وقتی object در جایگاه primitive لازم باشد، `[Symbol.toPrimitive]`(hint) → `toString()` → `valueOf()` صدا زده می‌شود.

</div>

```javascript
// x ToPrimitive سفارشی
const price = {
  amount: 1000,
  currency: 'IRR',
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return this.amount;
    if (hint === 'string') return `${this.amount} ${this.currency}`;
    return this.amount; // default
  },
};

console.log(+price);      // 1000 (number hint)
console.log(`${price}`);  // "1000 IRR" (string hint)
console.log(price + '');  // "1000 IRR" (default hint → number → string)

// Symbol.toStringTag
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }
  get [Symbol.toStringTag]() {
    return `Money<${this.currency}>`;
  }
}

console.log(Object.prototype.toString.call(new Money(100, 'USD')));
// "[object Money<USD>]"
```

<div dir="rtl">

‏تأثیر `toStringTag` در React: وقتی خطا می‌گیری، `Object.prototype.toString.call(component)` در DevTools نمایش داده می‌شود.

</div>

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏| سناریو | Coercion | ریسک |
‏|--------|----------|------|
‏| `if (value)` | Boolean coercion | `0`, `''`, `null`, `undefined`, `NaN` → false |
‏| `a == null` | Loose equality | فقط null/undefined را چک می‌کند (safe) |
‏| `+str` → number | Unary + | `+'abc'` → `NaN` (بیصدا) |
‏| `'' + num` → string | String coercion | آرام و predictable |
‏| `a == b` | Abstract Equality | الگوریتم ۷ مرحله‌ای پیچیده — `[] == ![]` → true |

</div>

```javascript
// x WTF moments — بدفهمی‌های رایج
[] == ![];    // true  — !!??
[] == 0;      // true
'' == 0;      // true
'\t' == 0;    // true
'\n' == 0;    // true
[1] == 1;     // true
[1,2] == NaN; // false (NaN === NaN هم false)

// x همه با === حل می‌شوند
[] === ![];   // false
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏**سناریو:** یک سیستم فاکتورینگ بین‌المللی که قیمت‌ها را از APIهای مختلف با فرمت‌های متفاوت می‌گیرد — بعضی string با کاما ("1,234.56")، بعضی number، بعضی object با amount/currency.

‏**چالش:** Implicit coercion در عملیات حسابی باعث `NaN` propagation می‌شود — یک API که `price: "1,234.56"` برگرداند، کل گزارش را خراب می‌کند.

‏**راه‌حل:** Normalizer با assertion قبل از هر عملیات حسابی.

</div>

```javascript
// Type-safe number normalizer
function normalizePrice(input) {
  if (typeof input === 'number' && Number.isFinite(input)) return input;
  if (typeof input === 'string') {
    const cleaned = input.replace(/[,$\s]/g, '');
    const num = Number(cleaned);
    if (!Number.isFinite(num)) throw new TypeError(`Invalid price: ${input}`);
    return num;
  }
  if (input?.amount !== undefined) return normalizePrice(input.amount);
  throw new TypeError(`Cannot coerce to price: ${JSON.stringify(input)}`);
}

// x Calculator — هر ورودی را assertion می‌کند
class MoneyCalculator {
  static sum(prices) {
    let total = 0;
    for (const p of prices) {
      const n = normalizePrice(p);
      if (Number.isNaN(n)) continue; // skip, not crash
      total += n;
    }
    return total;
  }
}

// x JSON.parse reviver — جلوگیری از string mutation
const reviver = (key, value) => {
  if (['price', 'total', 'amount'].includes(key) && typeof value === 'string') {
    return normalizePrice(value);
  }
  return value;
};
```

<div dir="rtl">

‏**نکته Performance:** `Number.isFinite()` بهتر از `isNaN()` است چون `NaN`, `Infinity`, `-Infinity` را همه reject می‌کند. `isNaN('abc')` → true (coercion می‌کند) اما `Number.isNaN('abc')` → false (بدون coercion).

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏برای جلوگیری از implicit coercion:

</div>

| راهکار | روش | محدودیت |
|--------|-----|---------|
| `===` همیشه | به جای `==` | `== null` را نمی‌توانی (safe pattern) |
| TypeScript strict mode | compile-time type check | runtime هنوز coercion دارد |
| `Object.is()` | مثل `===` + `-0` و `NaN` را درست هندل می‌کند | verbose |
| zod / io-ts | runtime validation | وابستگی, overhead |

<div dir="rtl">

‏**مقایسه Next.js vs Express.js:** در Next.js Server Components، داده‌ها در مرز server/client serialized می‌شوند — API response ممکن است numberهایی که به string تبدیل شده‌اند را برگرداند (JSON خودش number را حفظ می‌کند، اما APIهای شخص ثالث ممکن است string برگردانند). در Express.js، middleware مثل `body-parser` عددها را JSON.parse می‌کند اما objectIdهای MongoDB (ObjectID یا string) نیاز به attention دارند.

</div>

<div dir="rtl">

‏> **نهایی:** مهم‌ترین نکته Senior: از `===` به عنوان default استفاده کن. تنها جای مجاز `==`: `x == null` (چک کردن null و undefined با هم). برای عددهای API ورودی، همیشه explicit coercion با validation بزن — `+value` در production امن نیست چون `+'abc'` → `NaN` بیصدا. `Number.isFinite` دوست توست.

</div>
