<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "call, apply, bind (و پیاده‌سازی bind ساده با Closure)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏سه متد روی `Function.prototype` که `this` را در زمان فراخوانی تابع (call-time) به یک شیء دلخواه متصل می‌کنند:

</div>

```javascript
const obj = { name: "Ali" };

function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

// call(thisArg, arg1, arg2, …)
greet.call(obj, "Hello", "!");    // "Hello, Ali!"

// apply(thisArg, [argsArray])
greet.apply(obj, ["Hello", "!"]); // "Hello, Ali!"

// bind(thisArg, arg1, arg2, …) → returns new function
const boundGreet = greet.bind(obj, "Hello");
boundGreet("!");                   // "Hello, Ali!"
```

<div dir="rtl">

‏### تفاوت‌های فنی در سطح ECMAScript

‏- **`call`**: آرگومان‌ها را به صورت لیست (spread) می‌گیرد. `[[Call]]` را با `thisArg` تنظیم‌شده اجرا می‌کند.
‏- **`apply`**: آرگومان‌ها را به صورت آرایه می‌گیرد. از `[[Call]]` استفاده می‌کند با `CreateListFromArrayLike` روی `argsArray`.
‏- **`bind`**: یک تابع جدید (exotic object) با `[[BoundTargetFunction]]`، `[[BoundThis]]` و `[[BoundArguments]]` می‌سازد. وقتی تابع برگشتی صدا زده می‌شود، `[[BoundThis]]` را به عنوان `this` و `[[BoundArguments]]` را + آرگومان‌های جدید به تابع اصلی پاس می‌دهد.

</div>

```javascript
// x پیاده‌سازی دستی bind برای درک عمیق
Function.prototype.myBind = function (thisArg, ...boundArgs) {
  const originalFn = this; // تابعی که bind روی آن صدا زده شده

  // x چک: اگر this不是一个تابع باشد → خطا
  if (typeof originalFn !== "function") {
    throw new TypeError("Bind must be called on a function");
  }

  return function (...args) {
    // x ترکیب آرگومان‌های bind-time + call-time
    return originalFn.apply(thisArg, [...boundArgs, ...args]);
  };
};

// x استفاده
const boundMy = greet.myBind(obj, "Hello");
boundMy("!"); // "Hello, Ali!"
```

<div dir="rtl">

‏**در سطح V8:** وقتی `bind` صدا زده می‌شود، V8 یک `BoundFunction` می‌سازد که target function, bound this و bound args را ذخیره می‌کند. تابع برگشتی `prototype` ندارد (برخلاف توابع معمولی). اگر تابع اصلی `new.target` داشته باشد، `[[Construct]]` روی BoundFunction صدا زده می‌شود و `[[BoundTargetFunction]]` به عنوان constructor اجرا می‌شود — اما `[[BoundThis]]` نادیده گرفته می‌شود.

</div>

```javascript
function Point(x, y) {
  this.x = x;
  this.y = y;
}
const BoundPoint = Point.bind({ ignore: true }, 10);
const p = new BoundPoint(20);
console.log(p.x);             // 10
console.log(p.y);             // 20
console.log(p.ignore);        // undefined — [[BoundThis]] در new نادیده گرفته می‌شود
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **call** | **apply** | **bind** |
|------------|----------|-----------|-----------|
| **تعداد آرگومان مشخص** | ✅ | ❌ (overkill) | ❌ |
| **تعداد آرگومان متغیر** (مثلاً Math.max) | ❌ | ✅ با spread ریسک overflow دارد | ❌ |
| **Need stable reference** (مثلاً useEffect/removeEventListener) | ❌ | ❌ | ✅ |
| **Partial application** (پیش‌تنظیم آرگومان) | ❌ | ❌ | ✅ |
| **Method borrowing** (مثلاً `Array.prototype.slice.call`) | ✅ | ✅ interchangeable | ❌ (uneeded) |
| **Performance** | ✅ سریع‌ترین | ⚠️ کمی کندتر (args array) | ⚠️ ایجاد تابع جدید (هزینه یک بار) |
| **Inheritance (ساخت شیء با another prototype)** | ✅ `Object.create` بهتر است | ❌ | ❌ |

</div>

<div dir="rtl">

‏### قانون Senior

‏۱. **Method borrowing** → `call` (مثل `Object.prototype.toString.call(x)`)
‏۲. **آرگومان‌های متغیر** → بقیه تبدیل به spread operator شده‌اند (مدرن‌تر)
‏۳. **Partial application + stable reference** → `bind`
‏۴. **Performance-critical loops** → از `apply` با آرایه بزرگ خودداری کن (spread operator معادل است و optimize شده)
‏۵. **TypeScript** با `noImplicitAny: true` جلوی خیلی از خطاهای `this` در method borrowing را می‌گیرد

</div>

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: Performance Degradation با bind در React Components

‏در یک پروژه Enterprise با دیتای لوکیشن، تیم از `bind` داخل render برای جلوگیری از execution استفاده کرده بود:

</div>

```jsx
// x ***** ANTI-PATTERN: bind در render *****
const MarkerList = ({ markers, onSelect }) => {
  return (
    <div>
      {markers.map(m => (
        <Marker
          key={m.id}
          data={m}
          // x هر بار که render اجرا می‌شود، یک تابع جدید با bind ساخته می‌شود
          // x === عین arrow function در render
          onClick={onSelect.bind(null, m.id)}
        />
      ))}
    </div>
  );
};
```

<div dir="rtl">

‏**مشکل:** ۱۰۰۰ marker, ۱۰ بار re-render در ثانیه → ۱۰٬۰۰۰ تابع جدید در ثانیه. Garbage collector مدام active. UI lag.

</div>

```javascript
// x اثبات مصرف حافظه
class Sim extends React.Component {
  constructor(props) {
    super(props);
    this.handle = this.handle.bind(this); // ✅ یک بار در constructor: optimal
  }
  handle() { ... }
  render() {
    // x bind در render: this.handle.bind(this) → تابع جدید
    return <Child onClick={this.handle.bind(this)} />;
  }
}
```

<div dir="rtl">

‏**راه‌حل Production — استراتژی Senior:**

</div>

```jsx
// x ***** OPTIMAL: stable callback با useCallback *****
const MarkerList = ({ markers, onSelect }) => {
  // x Stable callback: یک بار ساخته می‌شود
  const handleSelect = useCallback((id) => {
    onSelect(id);
  }, [onSelect]); // فقط زمانی عوض می‌شود که onSelect عوض شود

  return (
    <div>
      {markers.map(m => (
        <Marker
          key={m.id}
          data={m}
          // x stable reference — اگر Marker با React.memo wraps شده باشد، re-render نمی‌کند
          onClick={handleSelect}
          id={m.id}
        />
      ))}
    </div>
  );
};

// x Marker با React.memo
const Marker = React.memo(({ data, onClick, id }) => {
  return <div onClick={() => onClick(id)}>{data.name}</div>;
});
```

<div dir="rtl">

‏**نکته‌ای که Junior نمی‌فهمد:**
‏- `bind` در constructor (در class components) فقط یک بار execute می‌شود — optimal
‏- `bind` در render معادل `() => fn()` است — هربار تابع جدید
‏- در Function Components هیچ دلیلی برای `bind` وجود ندارد — `useCallback` + closure جایگزین کامل است
‏- در **Micro-frontend** با Module Federation، `bind` روی توابع remote ممکن است due به isolated scope کار نکند — باید از proxy functions استفاده کرد

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزینه</div>

<div dir="rtl">

‏اگر React/Next.js نبود و در یک پروژه **Vanilla JS** یا **Vue.js** بودم:

</div>

<div dir="ltr">

| **نیاز** | **React callback** | **Vanilla JS** | **Vue.js** |
|----------|-------------------|----------------|------------|
| Method borrowing | `.call()` | `call/apply` | نیاز نیست (methodها روی Vue instance auto-bind) |
| Stable callback | `useCallback` | Closure variable | `.bind` یا closure |
| Partial application | Closure | `bind` با partial args | Closure یا computed |
| Event listener (dynamic `this`) | نیاز نیست | `bind`/arrow | `$event` در template |

</div>

<div dir="rtl">

‏**برنده:** **Closure** در همه موارد جایگزین `bind` می‌شود.

</div>

```javascript
// x جایگزین bind با closure — بهینه‌تر و قابل فهم‌تر
function createHandler(obj, prefix) {
  // closure replaces bind
  return function (msg) {
    return `${prefix}: ${msg}`;
  }.bind(obj); // bind اینجا redundant است چون closure از obj دارد
}

// x با closure خالص — بدون bind
function createHandlerPure(obj, prefix) {
  return function (msg) {
    return `${prefix}: ${msg}`;
  };
}
```

<div dir="rtl">

‏> **نکته:** در محیط‌هایی که V8 optimize شده (Node.js 14+, all modern browsers)، `function.call` سریع‌تر از spread operator است. برای hot paths از `call` استفاده کنید.

</div>

```javascript
// Hot path optimization
function fastSlice(array) {
  return Array.prototype.slice.call(array);
  // vs
  // x return [...array]; // spread در hot path ۱۰-۲۰٪ کندتر است
}
```

<div dir="rtl">

‏> **پس از تحویل این مبحث، ادامه بده.**

</div>