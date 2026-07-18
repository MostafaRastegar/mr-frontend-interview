<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "تفاوت عمیق == vs === — الگوریتم Abstract Equality Comparison" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**`===` (Strict Equality):** مقایسه را بدون هیچ تبدیل نوعی انجام می‌دهد. اگر نوع (Type) دو عملوند متفاوت باشد، بلافاصله `false` برمی‌گرداند. اگر نوع یکی بود، آن‌گاه سراغ SameValueZero یا SameValueNonNumeric می‌رود.

‏**`==` (Abstract Equality):** اگر نوع عملوندها متفاوت باشد، الگوریتم **Abstract Equality Comparison** (`ToPrimitive`, `ToNumber`, `ToObject`) را اجرا می‌کند تا نوع‌ها را همسان کند، سپس مقایسه می‌کند.

‏### هسته در سطح مشخصات ECMAScript (ES2015+)

‏الگوریتم کامل در `7.2.14 Abstract Equality Comparison`:

</div>

```
1. If Type(x) === Type(y) → return SameValueNonNumeric(x, y) — دقیقاً مثل ===
2. If x is null and y is undefined (یا برعکس) → return true
3. If Type(x) is Number and Type(y) is String → return x == ToNumber(y)
4. If Type(x) is String and Type(y) is Number → return ToNumber(x) == y
5. If Type(x) is Boolean → return ToNumber(x) == y
6. If Type(y) is Boolean → return x == ToNumber(y)
7. If Type(x) is either String/Number/BigInt and Type(y) is Object → return x == ToPrimitive(y)
8. If Type(x) is Object and Type(y) is String/Number/BigInt → return ToPrimitive(x) == y
9. Otherwise → return false
```

<div dir="rtl">

‏**نکته V8:** الگوریتم در C++ پیاده‌سازی شده در `src/objects/objects.cc` به عنوان `Object::Equals`. برای Objectها، اگر هر دو Object باشند، comparison pointer-based است (همان آدرس حافظه). `===` در `ic/i386/ic-compiler.cc` از طریق `CompareIC` با `STRICT` handler inline می‌شود.

‏### موارد خاص SameValueZero (Object.is)

‏- **`NaN === NaN`** → `false` (IEEE 754)
‏- **`+0 === -0`** → `true` (طبق SameValueNonNumeric)
‏- **`Object.is(NaN, NaN)`** → `true`
‏- **`Object.is(+0, -0)`** → `false`

</div>

```javascript
// x کاربرد عملی SameValueZero در Set و Map
const s = new Set();
s.add(NaN);
s.add(NaN); // Set فقط یک بار نگه می‌دارد — چون از SameValueZero استفاده می‌کند
console.log(s.size); // 1
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **Strict ===** | **Abstract ==** | **دلیل Senior** |
|------------|----------------|-----------------|------------------|
| **Validation فرم** (مقایسه `"5"` با `5`) | ❌ باید صریح تبدیل کنی | ✅ `==` کار می‌کند | اما باز هم `===` بهتر است: صریح‌تر و قابل پیش‌بینی |
| **بررسی nullable** (`null` یا `undefined`) | `x !== null && x !== undefined` | `x != null` (یک خط) | ✅ `== null` تنها کاربرد acceptable `==` در Enterprise |
| **Type-safe codebase** (TypeScript strict) | ✅ همیشه | ❌ غیرقابل پیش‌بینی | ESLint `eqeqeq: error` با `allow-null` استثنا |
| **JSON.parse مقادیر** (string "true" vs boolean) | ✅ `===` نیاز به parse اضافی ندارد | می‌تواند `"true" == true` را اشتباه بفهمد | کوئت: Coercion سورپرایز |
| **API response validation** | ✅ صریح | ❌ Coercion خطاهای پنهان می‌سازد | `"false" == true` → `false` ولی `"false" == false` → `false` هم! |

</div>

<div dir="rtl">

‏### قانون سرانگشتی Senior

‏- در **کد Production**: همیشه `===` (فعال کردن `eqeqeq` در ESLint)
‏- تنها استثنا: **`x == null`** به جای `x === null || x === undefined`
‏- در **TypeScript strict mode**: `==` عملاً خطای کامپایلر می‌دهد مگر `null`

</div>

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: Coercion پنهان در Validation API

‏یک سناریوی واقعی در پروژه‌ای با معماری Monorepo (۵ تیم مختلف، ۳ Stack):

‏تیم Payment API یک فیلد `amount` را به صورت `number` در JSON برمی‌گرداند. تیم UI یک Input داشت که کاربر `amount` را به صورت string تایپ می‌کرد. یک تیم Junior از `==` برای مقایسه استفاده کرد:

</div>

```javascript
// x کد ارسالی Junior — به نظرش درست می‌آمد چون `"100" == 100` true است
function validatePayment(amount, threshold) {
  // x amount از API به صورت number می‌آید
  // x threshold از Input کاربر به صورت string می‌آید
  if (amount == threshold) {    // ← اینجا خطرناک است
    return { valid: true };
  }
  return { valid: false };
}

// x تست‌ها پاس می‌شد:
console.log(validatePayment(100, "100")); // true ✅
console.log(validatePayment(0, "0"));     // true ✅

// x ولی در Production:
console.log(validatePayment(null, "null"));   // false — چون "null" == null false است
console.log(validatePayment(undefined, ""));  // false — undefined == "" false
console.log(validatePayment(false, "0"));     // true  ← BUG! false == "0" برابر true است
// x چرا؟ false → ToNumber(false) = 0 ; "0" → ToNumber("0") = 0 ; 0 == 0 → true
```

<div dir="rtl">

‏**راه‌حل Production:**

</div>

```javascript
function validatePayment(amount, threshold) {
  // x ابتدا type guarding صریح
  if (typeof amount !== 'number' || typeof threshold !== 'string') {
    throw new TypeError('Invalid input types');
  }
  
  const numericThreshold = parseInt(threshold, 10);
  if (isNaN(numericThreshold)) {
    return { valid: false, error: 'Invalid threshold format' };
  }
  
  // x === صریح — بدون سورپرایز
  if (amount === numericThreshold) {
    return { valid: true };
  }
  return { valid: false, error: `Threshold ${numericThreshold} not matched` };
}
```

<div dir="rtl">

‏**نکته کلیدی که Junior نمی‌داند:**
‏۱. `==` در ۲۸ حالت مختلف می‌تواند true برگرداند که ۱۲ تای آن غیرشهودی است
‏۲. **`[] == ![]`** → `true` است (چون `![]` → `false` → `0` ; `[]` → `""` → `0`)
‏۳. در TypeScript strict mode با `strictNullChecks`، `==` فقط روی `null/undefined` اجازه دارد — بقیه خطا است

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏اگر JavaScript نبود و مجبور بودم از **Python** یا **Rust** استفاده کنم:

</div>

<div dir="ltr">

| **JS: `==` vs `===`** | **Python** | **Rust** |
|------------------------|------------|----------|
| `"5" == 5` → `true` (coercion) | `"5" == 5` → `False` (هیچگاه coercion) | کامپایل می‌شکند — نوع‌ها متفاوت |
| `null == undefined` → `true` | `None is None` (نه ==) | `Option::None` |
| `0 == false` → `true` | `0 == False` → `True` (این هم coercion دارد) | کامپایل می‌شکند |
| **Coercion؟** | فقط بین عدد و bool | هرگز |

</div>

<div dir="rtl">

‏### Next.js vs Python/Rust — کدام برتری دارد؟

‏مقایسه `==` و `===` صرفاً مربوط به JavaScript است. در Next.js، TypeScript strict mode عملاً این بحث را حل کرده: ESLint `eqeqeq: error` + Prettier خودکار تبدیل می‌کند.

‏- **Python** با `==` هنوز بین `0 == False` سورپرایز دارد
‏- **Rust** با سیستم type قوی، این بحث را به طور کامل حذف کرده — **برنده مطلق**
‏- **Next.js + TypeScript + ESLint** شکل عملی برنده در اکوسیستم Frontend است چون سطحی از safety را با ابزارهای موجود فراهم می‌کند

</div>

---

<div dir="rtl">

‏> **پس از تحویل این مبحث، منتظر تأیید من بمان. پس از تأیید، طبق دستورات قسمت Automation Instructions پیش برو و سپس متوقف شو.**

</div>