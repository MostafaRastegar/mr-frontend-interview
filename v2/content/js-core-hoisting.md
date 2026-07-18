<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Hoisting در var, let, const و function declarations — منطقه مرده زمانی (TDZ)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Hoisting** رفتاری در JavaScript است که در فاز compile-time (قبل از اجرای خط‌به‌خط)، اعلان‌ها (declarations) را به بالای scope منتقل می‌کند. اما مقداردهی (initialization) در جای خود باقی می‌ماند.

‏### مکانیزم دقیق در سطح مشخصات ECMAScript

‏هر Execution Context دو فاز دارد:
‏۱. **Creation Phase** (فاز ایجاد): scopeها ساخته می‌شوند، variable/function declarations ثبت می‌شوند.
‏۲. **Execution Phase** (فاز اجرا): کد خط به خط اجرا می‌شود.

‏#### رفتار سه‌گانه در Creation Phase:

</div>

```
┌─────────────────┬──────────────┬──────────────┬──────────────────┐
│    Declaration  │ Hoisted?     │ Initialized  │ TDZ (Temporal    │
│                 │ (Scope Known)│ to?          │ Dead Zone)?      │
├─────────────────┼──────────────┼──────────────┼──────────────────┤
│ var             │ ✅ Yes       │ undefined    │ ❌ No            │
│ let             │ ✅ Yes       │ uninit.      │ ✅ Yes           │
│ const           │ ✅ Yes       │ uninit.      │ ✅ Yes           │
│ function decl.  │ ✅ Yes       │ function body│ ❌ No            │
│ class decl.     │ ✅ Yes       │ uninit.      │ ✅ Yes           │
└─────────────────┴──────────────┴──────────────┴──────────────────┘
```

<div dir="rtl">

‏**نکته کلیدی:** `let` و `const` **hoist می‌شوند** — برخلاف باور عام اشتباه. تفاوت در initial value است: `var` بلافاصله به `undefined` مقداردهی می‌شود، اما `let/const` در وضعیت `uninitialized` می‌مانند تا به خط مقداردهی برسند. دسترسی به آن‌ها قبل از خط مقداردهی، `ReferenceError` می‌دهد — این ناحیه را **Temporal Dead Zone (TDZ)** می‌نامند.

‏### تایید Hoisting شدن let/const در V8

</div>

```javascript
// x این خط ReferenceError می‌دهد، نه ReferenceError: x is not defined
// x اگر let/const hoist نمی‌شدند، خطا می‌داد: "x is not defined"
// x ولی خطا می‌دهد: "Cannot access 'x' before initialization" — یعنی scope می‌شناسدش
{
  console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = 5;
}
```

<div dir="rtl">

‏**در سطح V8:** در فاز Parsing، اسکوپ‌ها ساخته می‌شوند. برای `let/const`، slot در Environment Record رزرو می‌شود اما مقدار `uninitialized_marker` می‌گیرد. تا وقتی که خط مقداردهی اجرا نشود، دسترسی به آن slot خطای TDZ می‌دهد. برای `var`، مقدار `undefined` از ابتدا در slot قرار می‌گیرد.

</div>

```javascript
// x اثبات: let در try/catch با typeof هم ReferenceError می‌دهد
function demo() {
  typeof x;       // "undefined" — x حتی وجود خارجی ندارد
  typeof y;       // ReferenceError — y در TDZ است (hoisted ولی uninitialized)
  let y = 1;
}
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **var** | **let** | **const** |
|------------|---------|---------|-----------|
| **Hoisting safety** | ❌ `undefined` خاموش — باگ پنهان | ✅ TDZ — خطای زودهنگام | ✅ TDZ — خطای زودهنگام |
| **Reassignment** | ✅ مجاز | ✅ مجاز | ❌ ممنوع |
| **Block scope** | ❌ function-scoped | ✅ block-scoped | ✅ block-scoped |
| **Global pollution** | ❌ به `window` اضافه می‌کند | ✅ به `window` اضافه نمی‌کند | ✅ به `window` اضافه نمی‌کند |
| **Loop closure bug** | ❌ مشکل `for (var i...)` | ✅ حلقه‌ای مستقل | ✅ (برای ثابت‌ها) |
| **Enterprise rule** | ❌ ممنوع | ✅ پیش‌فرض | ✅ پیش‌فرض برای immutable |

</div>

<div dir="rtl">

‏### قانون Senior در Enterprise

‏- **`const` پیش‌فرض** — مگر نیاز به reassign داشته باشی که آن‌گاه `let`
‏- **`var` ممنوع** — ESLint `no-var: error`
‏- در TypeScript با `strict: true`، `let` رفتار TDZ را حفظ می‌کند
‏- در **loops**: `for (const x of arr)` کار می‌کند چون هر iteration یک binding جدید می‌سازد

</div>

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: Hoisting در Legacy Code Migration

‏پروژه‌ای با ۲۰۰٬۰۰۰ خط کد jQuery-era که به React/Next.js مهاجرت می‌کند. تیم ۱۵ نفره. کد قدیمی پر از `var` و hoisting dependencies ضمنی است.

‏**مشکل:** یک ماژول قدیمی که با `var` ساخته شده بود، وابستگی‌هایش را از طریق hoisting تأمین می‌کرد:

</div>

```javascript
// x ***** فایل قدیمی: utils/old-payment.js *****
function processPayment(amount) {
  // x اینجا از helperValidator استفاده می‌کند — که بعد از این تابع تعریف شده
  return helperValidator(amount) * TAX_RATE;  // هم helperValidator و هم TAX_RATE hoist می‌شوند
}

// x صدها خط بعد...
var TAX_RATE = 1.09;   // hoist به undefined اولیه، بعداً مقدار می‌گیرد
function helperValidator(amt) {  // function declaration کامل hoist می‌شود
  return amt > 0 ? amt : 0;
}
```

<div dir="rtl">

‏تیم Junior در refactoring تصمیم گرفت `var` را به `const` تبدیل کند (که کار درستی است) اما متوجه **order-dependency hoisting** نشد:

</div>

```javascript
// x ***** Refactored توسط Junior — BROKEN *****
function processPayment(amount) {
  return helperValidator(amount) * TAX_RATE;  // ReferenceError: TAX_RATE در TDZ
}

// x صدها خط بعد...
const TAX_RATE = 1.09;   // دیگر hoist نمی‌شود به undefined — در TDZ می‌ماند
function helperValidator(amt) {  // function declaration هنوز کامل hoist می‌شود
  return amt > 0 ? amt : 0;
}
```

<div dir="rtl">

‏**راه‌حل Production — مهاجرت مرحله‌ای:**

</div>

```javascript
// x ***** راه‌حل Senior: مرحله‌ای با Top-Level exports *****
// x ۱. ابتدا imports و همه function declarations را به بالای فایل ببر
// x ۲. const/let را به بالاترین خط ممکن (بعد از imports) ببر
// x ۳. با ESLint import/order ترتیب را اجباری کن

import { TAX_RATE } from './constants';  // حذف وابستگی محلی

function helperValidator(amt: number): number {
  return amt > 0 ? amt : 0;
}

function processPayment(amount: number): number {
  return helperValidator(amount) * TAX_RATE;
}

// x ۴. مرحله نهایی: کل ماژول به TypeScript strict mode — کامپایلر همه TDZ violations را می‌گیرد
```

<div dir="rtl">

‏**نکته کلیدی که Junior نمی‌داند:**
‏۱. `function declarations` هم hoist می‌شوند و هم body شان هم hoist می‌شود — برخلاف `function expressions` که فقط متغیر hoist می‌شود (`var` → `undefined`)
‏۲. `class declarations` TDZ دارند — `class expressions` هم بسته به نوع متغیرشان
‏۳. در **Module Pattern** (ES Modules)، هر ماژول strict mode دارد — `var` در strict mode باز هم به `globalThis` اضافه نمی‌شود

</div>

```javascript
// x تفاوت function declaration vs expression در hoisting
foo(); // ✅ "hello" — function declaration کامل hoist شده
bar(); // ❌ TypeError: bar is not a function — var bar به undefined است
function foo() { console.log('hello'); }
var bar = function() { console.log('world'); };
```

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزینه</div>

<div dir="rtl">

‏اگر JavaScript نبود و مجبور بودم از **Python** یا **Rust** استفاده کنم:

</div>

<div dir="ltr">

| **JS Hoisting/TDZ** | **Python** | **Rust** |
|----------------------|------------|----------|
| `var x; console.log(x);` → `undefined` | `print(x)` قبل از تعریف → `NameError` | کامپایل می‌شکند — استفاده قبل از تعریف |
| `let x; console.log(x);` → `ReferenceError` | `def f(): print(x); x=5` → خطای runtime (UnboundLocalError) | کامپایل می‌شکند |
| Hoisting مفهومی | هیچ نوع hoisting ندارد | همه bindingها قبل از استفاده تعریف شده باشند |
| TDZ مفهومی | وجود ندارد (از اول مشخص نیست) | Borrow checker مشابه TDZ عمل می‌کند |

</div>

<div dir="rtl">

‏**برنده:** Rust با borrow checker. در Rust، کامپایلر تضمین می‌کند که هیچ variableای قبل از initialization استفاده نمی‌شود — این قوی‌تر از TDZ است چون در سطح compile-time است نه runtime.

‏**Next.js + TypeScript** با `strict: true` و `noUnusedLocals` نزدیک‌ترین تجربه به Rust را می‌دهد: کامپایلر TDZ violations را می‌گیرد.

</div>

---

<div dir="rtl">

‏> **پس از تحویل این مبحث، منتظر تأیید من بمان. پس از تأیید، طبق دستورات قسمت Automation Instructions پیش برو و سپس متوقف شو.**

</div>