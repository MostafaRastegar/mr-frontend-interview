<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "this در ۴ حالت مختلف (Global, Function, Arrow Function, و در کلاس‌ها)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**`this`** یک کلمه کلیدی در JavaScript است که به شیء (object) context جریان اجرا اشاره می‌کند. مقدار `this` در زمان **فراخوانی تابع (call-time) تعیین می‌شود** — نه در زمان تعریف — مگر در arrow functions که در زمان تعریف binding می‌شوند.

‏### ۴ حالت اصلی در سطح ECMAScript

‏#### ۱. Global Context (در مرورگر و Node.js)

</div>

```javascript
// x در مرورگر (غیر strict mode)
console.log(this); // window

// x در Node.js (ماژول)
console.log(this); // module.exports (خالی: {})

// x در strict mode (چه مرورگر و چه Node.js)
"use strict";
console.log(this); // undefined
```

<div dir="rtl">

‏**در سطح V8:** در Global Environment Record، مقدار `[[ThisValue]]` برابر با `globalThis` است (در مرورگر `window`، در Node.js `global`). در ES Modules (همه فایل‌های `.mjs` یا `"type": "module"`) strict mode فعال است و `this` ماژول برابر `undefined` است.

</div>

#### <div dir="rtl">‏۲. Function Context (غیر arrow)</div>

```javascript
function showThis() {
  return this;
}

// x غیر strict mode
showThis();                    // globalThis (window/global)

// strict mode
"use strict";
showThis();                    // undefined

// x با شیء والد
const obj = { method: showThis };
obj.method();                  // obj

// x با new
new showThis();                // instance جدید — [[Construct]] this را به instance جدید تنظیم می‌کند
```

<div dir="rtl">

‏**در سطح ECMAScript:** تابع غیر arrow، `[[Call]]` داخلی را اجرا می‌کند که `this` را از `[[ThisMode]]` تابع می‌گیرد:
‏- `lexical` (arrow functions): `this` از محیط لغوی (outer scope)
‏- `strict`: `this` همان مقداری است که caller می‌دهد (یا `undefined`)
‏- `global`: `this` اگر `undefined` یا `null` باشد، به `globalThis` تبدیل می‌شود

</div>

#### <div dir="rtl">‏۳. Arrow Function (Lexical this)

</div>

```javascript
const obj = {
  name: "test",
  regular: function() {
    return this.name;
  },
  arrow: () => {
    return this.name;
  }
};

obj.regular(); // "test"
obj.arrow();   // undefined (یا مقدار this از scope بالاتر)

// x تفاوت حیاتی در DOM event handlers
button.addEventListener("click", function() {
  console.log(this); // button
});
button.addEventListener("click", () => {
  console.log(this); // window (یا undefined در strict mode)
});
```

<div dir="rtl">

‏**در سطح V8:** Arrow function در زمان تعریف، `[[ThisMode]]` را روی `lexical` تنظیم می‌کند. در زمان فراخوانی، `[[Call]]` این مقدار را نادیده می‌گیرد و `this` را از Environment Record بیرونی (outer lexical environment) می‌گیرد. بدیهتاً arrow function متد `[[Construct]]` ندارد — نمی‌توان با `new` صدا زد.

</div>

```javascript
// x اثبات: arrow function cannot be constructor
const Arrow = () => {};
new Arrow(); // TypeError: Arrow is not a constructor

// x اثبات: no prototype property
console.log(Arrow.prototype); // undefined
```

#### <div dir="rtl">‏۴. Class Context

</div>

```javascript
class User {
  constructor(name) {
    this.name = name; // this = instance جدید
  }

  greet() {
    return `Hello, ${this.name}`;
  }

  // x Class field (پیشنهادی) + arrow = دردسر!
  greetArrow = () => {
    return `Hello, ${this.name}`;
  };
}

const u = new User("Ali");
u.greet();       // "Hello, Ali"
u.greetArrow();  // "Hello, Ali"

// x مشکل: destructuring method
const { greet, greetArrow } = u;
greet();          // TypeError: Cannot read properties of undefined
greetArrow();     // "Hello, Ali" — چون lexically bound به instance
```

<div dir="rtl">

‏**در سطح V8 و Class Features:** کلاس‌ها در strict mode اجرا می‌شوند. بدنه کلاس (body) در strict mode است. Class fields با `Object.defineProperty` روی هر instance تعریف می‌شوند — نه روی prototype. arrow function در class field یک arrow است که در constructor زمان تعریف می‌شود و `this` را از scope constructor می‌گیرد (که همان instance جدید است).

</div>

```javascript
// x کلاس‌ها در strict mode — اثبات
class Strict {
  constructor() {
    console.log(this); // instance
    console.log(new.target); // کلاس (در فراخوانی با new)
  }
}
// x Strict() بدون new → TypeError: Class constructor Strict cannot be invoked without 'new'
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **regular function** | **arrow function** | **class method** |
|------------|----------------------|--------------------|-----------------------------|
| **Dynamic `this`** (مثلاً DOM events) | ✅ `this` = target | ❌ `this` از outer scope | ✅ با bind یا class field arrow |
| **Method chaining** | ✅ `return this` | ❌ (نمی‌توان `this` را عوض کرد) | ✅ `return this` |
| **Callback / Promise chain** | ❌ نیاز به `.bind()` یا closure | ✅ lexical this | ❌ نیاز به arrow callback یا bind |
| **Constructor / `new`** | ✅ قابل استفاده | ❌ خطا | ✅ اجباری با `new` |
| **prototype inheritance** | ✅ | ❌ | ✅ |
| **Performance** | ✅ یک بار روی prototype | ❌ per-instance copy | ✅ prototype (class field arrow: per-instance) |

</div>

<div dir="rtl">

‏### قانون Senior در Enterprise

‏۱. **متدهای کلاس** → روش پیش‌فرض (prototype-based)
‏۲. اگر متد به عنوان callback استفاده می‌شود (مثلاً React event handler) → **class field arrow**
‏۳. اگر نیاز به dynamic `this` دارید (مثلاً DOM event listener که نیاز به target دارد) → **regular function**
‏۴. در **FP (Functional Programming)** → arrow functions غالب
‏۵. **TypeScript** با `noImplicitThis: true` و `strict: true` خیلی از خطاهای `this` را در compile-time می‌گیرد

</div>

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: React Class Components Legacy Migration

‏ساخت ۲۰۱۹. تیم در حال مهاجرت از Class Components به Hooks. تیم ۲۰ نفره. یک کامپوننت حیاتی نقشه (MapViewer) که ۱۵۰۰ خط است و از `this` در ۱۲ جا استفاده می‌کند.

</div>

```jsx
// x ***** Legacy Component — قبل از مهاجرت *****
class MapViewer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { zoom: 5, markers: [] };
    // x اینجا سه روش مختلف برای binding this استفاده شده!
    this.handleZoom = this.handleZoom.bind(this);   // روش ۱: bind در constructor
  }

  handleZoom(level) { /* از this.state استفاده می‌کند */ }

  // x روش ۲: class field arrow (پیشنهادی ۲۰۲۰)
  handleMarkerClick = (id) => { /* از this.props استفاده می‌کند */ };

  render() {
    return (
      <div>
        {/* x روش ۳: arrow در render — ANTI-PATTERN! */}
        <button onClick={() => this.handleZoom(this.state.zoom + 1)}>
          Zoom In
        </button>
        <MarkerList
          markers={this.state.markers}
          // x مشکل: هر بار که render اجرا می‌شود، یک تابع جدید ساخته می‌شود
          onMarkerClick={(id) => this.handleMarkerClick(id)}
        />
      </div>
    );
  }
}
```

<div dir="rtl">

‏**مشکل:** Junior در refactoring به Hooks، arrow در render را بدون فکر به function component منتقل کرد:

</div>

```jsx
// x ***** Refactored توسط Junior — BROKEN *****
function MapViewer({ markers: initialMarkers }) {
  const [zoom, setZoom] = useState(5);
  const [markers, setMarkers] = useState(initialMarkers);

  // x فراموش کرد این رو refactor کنه — this وجود خارجی ندارد
  const handleZoom = (level) => {
    // x قبلاً اینجا this.setState بود — حالا چی؟!
    // x Junior نوشت: setZoom(this.state.zoom + level) // ReferenceError!
  };

  // x این یکی ماندگار است چون arrow بود — ولی setState ندارد
  const handleMarkerClick = (id) => {
    // x قبلاً this.setState بود — حالا?
  };

  // x مشکل: ۳ روش مختلف binding، همه شکسته
  return (
    <div>
      <button onClick={() => handleZoom(zoom + 1)}>Zoom In</button>
        {/* x همین طور each render new function */}
      <MarkerList markers={markers} onMarkerClick={(id) => handleMarkerClick(id)} />
    </div>
  );
}
```

<div dir="rtl">

‏**راه‌حل Production — استراتژی مهاجرت Senior:**

</div>

```jsx
// x ***** راه‌حل Senior: گام‌های مهاجرت سیستماتیک *****
// x گام ۱: شناسایی all `this` usage در کامپوننت (grep this.)
// x گام ۲: تبدیل `this.state.x` به state variables مستقیم
// x گام ۳: از useCallback برای جلوگیری از re-creation callbackها
// x گام ۴: test با React Testing Library بعد از هر تغییر

function MapViewer({ markers: initialMarkers }) {
  const [zoom, setZoom] = useState(5);
  const [markers, setMarkers] = useState(initialMarkers);
  // x NOTE: اینها از state مستقیم استفاده می‌کنند — نیاز به this نیست

  const handleZoom = useCallback((level) => {
    setZoom(prev => prev + level);
  }, []); // pure stable reference

  const handleMarkerClick = useCallback((id) => {
    setMarkers(prev => prev.map(m =>
      m.id === id ? { ...m, selected: !m.selected } : m
    ));
  }, []);

  // x 🐴 نکته کلیدی: if callback needs latest props/state, use ref
  const countRef = useRef(0);
  countRef.current = markers.length; // latest value بدون re-creation callback

  return (
    <div>
      <button onClick={() => handleZoom(1)}>Zoom In</button>
      <MarkerList markers={markers} onMarkerClick={handleMarkerClick} />
    </div>
  );
}
```

<div dir="rtl">

‏**خطایی که Junior نمی‌فهمد:**
‏۱. `this` در function component وجود ندارد — همه callbackها closure روی state variables می‌بندند
‏۲. `useCallback` از re-creation جلوگیری می‌کند — معادل `bind(this)` در constructor
‏۳. `useRef` برای دسترسی به latest value بدون شکستن referential identity
‏۴. در **Micro-frontend** با Module Federation، اشتراک `this` بین remoteها غیرممکن است — هر remote یک React copy دارد

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزینه</div>

<div dir="rtl">

‏اگر React/Next.js نبود و مجبور بودم از **Vue.js** یا **Svelte** استفاده کنم:

</div>

<div dir="ltr">

| **JS `this` behavior** | **Vue.js** | **Svelte** |
|------------------------|------------|------------|
| `this` in methods | ✅ Vue auto-binds methods | ❌ بدون `this` در `<script>` سطح بالا |
| `this` in arrow callbacks | ✅ دستی باید bind کرد | ✅ نیازی نیست (بدون `this`) |
| `this` in class components | Vue 2 Options API دارد | کامپوننت ندارند |
| Event handler `this` | ✅ `this` = Vue instance | ندارد (`on:click={handler}` مستقیم) |
| Complexity | میان — نیاز به فهمید `this` | کم — بدون `this` در template |

</div>

<div dir="rtl">

‏**برنده:** Svelte. در Svelte، `this` وجود تقریباً ندارد (در `<script>` از top-level variables استفاده می‌شود). منطق کامپوننت در closure کامپایل می‌شود — شبیه function components در React.

‏**Next.js + TypeScript** با `noImplicitThis` و `strictFunctionTypes` نزدیک‌ترین تجربه به Svelte را می‌دهد.

</div>

```tsx
// x Next.js App Router — بدون this
// x کامپوننت‌ها function هستند — this وجود ندارد
export default function Page() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

// x اگر نیاز به class دارید (قدیمی) — TypeScript strict کمک می‌کند
class OldComponent extends React.Component<Props, State> {
  handleClick = (): void => { // class field arrow — this درست است
    this.setState(prev => ({ count: prev.count + 1 }));
  };
}
```

<div dir="rtl">

‏> **پس از تحویل این مبحث، منتظر تأیید من بمان. پس از تأیید، طبق دستورات قسمت Automation Instructions پیش برو و سپس توقف شو.**

</div>