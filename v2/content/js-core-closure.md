

# Closure در JavaScript

مناسب برای مصاحبه فنی | تاریخ: 2026-07-19

---

## Closure چیست؟

یک **Closure** ترکیب یک تابع با محیط (environment) آن تابع است. به زبان ساده: وقتی یک تابع داخل تابع دیگری تعریف می‌شود، تابع داخلی به متغیرهای تابع بیرونی دسترسی دارد — حتی بعد از اینکه تابع بیرونی اجرایش تمام شده باشد.

```javascript
function outer() {
  const message = "سلام";
  
  function inner() {
    console.log(message); // inner به message دسترسی دارد
  }
  
  return inner;
}

const fn = outer();
fn(); // "سلام"
```

چرا این مهم است؟ تابع `outer` تمام شده و از Call Stack خارج شده، اما متغیر `message** هنوز در حافظه زنده است و `inner` می‌تواند به آن دسترسی داشته باشد. چون `inner` یک Closure روی `message` دارد.

---

## چرا Closure ایجاد می‌شود؟ (مکانیزم دقیق)

وقتی یک تابع در JavaScript ایجاد می‌شود، یک **Lexical Environment** دارد:
- **Local Scope**: متغیرهای داخل خود تابع
- **Outer Environment**: reference به محیط بیرونی (جایی که تابع تعریف شده، نه جایی که فراخوانی می‌شود)

وقتی تابعی return می‌شود و جایی ذخیره می‌شود، موتور JavaScript هیچ‌کدام از این متغیرها را پاک نمی‌کند. چون تابع بازگشت‌شده هنوز به آن‌ها reference دارد.

```
outer() اجرا می‌شود
┌─────────────────────────────┐
│ Lexical Environment:        │
│   message = "سلام"          │
│   inner = [function ref]    │
│   outerEnv = → Global      │
└──────────┬──────────────────┘
           │
           │ return inner
           ↓
┌─────────────────────────────┐
│ fn = inner (با closure)     │
│   ← هنوز به message وصل است│
└─────────────────────────────┘
```

---

## کاربردهای اصلی Closure در مصاحبه

### ۱. داده خصوصی (Private State / Encapsulation)

```javascript
function createCounter() {
  let count = 0; // خصوصی — از بیرون دسترسی مستقیم نداری

  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
console.log(counter.count);       // undefined (خصوصی!)
```

### ۲. Function Factory

```javascript
function createMultiplier(factor) {
  return function (number) {
    return number * factor;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

هر بار `createMultiplier` فراخوانی می‌شود، یک Closure جدید با `factor` خاص خودش ساخته می‌شود.

### ۳. Debounce (کاربرد واقعی در پروژه)

```javascript
function debounce(fn, delay) {
  let timerId;

  return function (...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

const search = debounce((query) => {
  console.log(`جستجو: ${query}`);
}, 300);

// هر بار search فراخوانی شود، timerId قبلی را پاک می‌کند
```

### ۴. Loop Problem (سوال رایج مصاحبه)

```javascript
// مشکل经典的 — var در loop
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// خروجی: 3 → 3 → 3 (نه 0 → 1 → 2)

// راه‌حل ۱: let (block scope)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// خروجی: 0 → 1 → 2

// راه‌حل ۲: IIFE با closure
for (var i = 0; i < 3; i++) {
  (function (j) {
    setTimeout(() => console.log(j), 100);
  })(i);
}
// خروجی: 0 → 1 → 2
```

تفاوت `var` و `let` در loop: `var` function-scoped است و همه closureها به یک متغیر `i` وصل‌اند. `let` block-scoped است و هر iteration یک binding جدید می‌سازد.

---

## Module Pattern (سناریوی واقعی)

```javascript
const PaymentModule = (function () {
  let balance = 0; // خصوصی
  const transactions = []; // خصوصی

  return {
    deposit(amount) {
      balance += amount;
      transactions.push({ type: 'deposit', amount, date: Date.now() });
    },
    getBalance() {
      return balance;
    },
    getHistory() {
      return [...transactions]; // کپی — نه reference اصلی
    },
  };
})();

PaymentModule.deposit(100);
PaymentModule.deposit(50);
console.log(PaymentModule.getBalance()); // 150
```

این الگو قبل از ES6 Modules رایج بود. امروزه از ES Modules استفاده می‌شود، اما درک این الگو نشان‌دهنده درک عمیق Closure است.

---

## Closure و Memory

هر closure یک reference به متغیرهای محیط بیرونی دارد. اگر closure به متغیر بزرگی وصل باشد، آن متغیر تا زمانی که closure زنده است در حافظه می‌ماند.

```javascript
function processData() {
  const hugeData = new Array(1000000).fill('x'); // ۱ میلیون آیتم
  
  return function summarize() {
    return hugeData.length; // hugeData همیشه در حافظه می‌ماند
  };
}

const summarize = processData(); // hugeData پاک نمی‌شود
console.log(summarize()); // 1000000
```

**نکته مهم:** اگر واقعاً فقط به `length` نیاز دارید، مقدار را در closure ذخیره کنید نه کل آرایه:

```javascript
function processData() {
  const hugeData = new Array(1000000).fill('x');
  const length = hugeData.length; // فقط عدد — حافظه کمتر
  // hugeData حالا می‌تواند GC شود

  return function summarize() {
    return length;
  };
}
```

---

## سوالات رایج مصاحبه

| سوال | پاسخ کوتاه |
|------|------------|
| Closure چیست؟ | تابع + محیط لغوی آن. تابعی که به متغیرهای scope بیرونی‌اش دسترسی دارد حتی بعد از اتمام اجرای scope بیرونی. |
| چه چیزی در closure نگه داشته می‌شود؟ | Reference به متغیرها — نه کپی ارزش‌ها. اگر متغیر تغییر کند، closure مقدار جدید را می‌بیند. |
| تفاوت closure با scope چیست؟ | Scope محدوده دسترسی است. Closure وقتی ایجاد می‌شود که تابعی از scope خودش خارج شود و هنوز به متغیرهای آن scope دسترسی داشته باشد. |
| Closure چه تأثیری بر memory دارد؟ | متغیرهایی که به آن‌ها reference دارد تا زمان حیات closure در حافظه می‌مانند. می‌تواند memory leak ایجاد کند اگر بی‌احتیاط باشید. |
| `bind` چگونه با closure کار می‌کند؟ | `bind` یک تابع جدید برمی‌گرداند که `this` و partial arguments را در خود ذخیره کرده — این خودش closure است. |

---

## پیاده‌سازی دستی bind (سؤال مصاحبه‌ای معروف)

```javascript
Function.prototype.myBind = function (context, ...boundArgs) {
  const originalFn = this;

  return function (...callArgs) {
    return originalFn.apply(context, [...boundArgs, ...callArgs]);
  };
};

function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const obj = { name: "علی" };
const bound = greet.myBind(obj, "سلام");
console.log(bound("!")); // "سلام، علی!"
```

`originalFn` و `context` و `boundArgs` همه در closure ذخیره شده‌اند و وقتی `bound` فراخوانی می‌شود، به آن‌ها دسترسی دارد.

---

> **پس از تحویل این مبحث، منتظر تأیید من بمان. پس از تأیید، طبق دستورات قسمت Automation Instructions پیش برو و سپس متوقف شو.**
