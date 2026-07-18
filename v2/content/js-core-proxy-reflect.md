<div dir="rtl">

شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

لطفاً سوال "Proxy و Reflect (کاربرد در Validation، Trapping، و Reactive Programming - مثل Vue.js)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

**Proxy** object ای است که عملیات روی یک object دیگر (target) را intercept می‌کند. هر operation (خواندن، نوشتن، حذف، فراخوانی) می‌تواند توسط trap handler قبل از اجرا یا بعد از آن تغییر کند.

**Reflect** API ای است که عملیات object را به صورت default اجرا می‌کند — معمولاً داخل Proxy trap استفاده می‌شود تا behavior پیش‌فرض حفظ شود.

</div>

```javascript
// x Proxy ساده
const handler = {
  get(target, prop, receiver) {
    console.log(`reading ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    console.log(`writing ${prop} = ${value}`);
    return Reflect.set(target, prop, value, receiver);
  }
};

const proxy = new Proxy({ name: "ali" }, handler);
proxy.name;            // "reading name" → "ali"
proxy.age = 25;        // "writing age = 25"

// Reflect — default behavior
const obj = { x: 1 };
Reflect.get(obj, "x"); // 1
Reflect.has(obj, "x"); // true
```

<div dir="rtl">

### Proxy Traps (۱۳ trap)

</div>

```javascript
// x تمام traps
const handler = {
  get(target, prop, receiver) {},           // خواندن property
  set(target, prop, value, receiver) {},     // نوشتن property
  has(target, prop) {},                      // operator in
  deleteProperty(target, prop) {},           // operator delete
  apply(target, thisArg, args) {},           // فراخوانی function
  construct(target, args) {},                // new operator
  getPrototypeOf(target) {},                 // Object.getPrototypeOf
  setPrototypeOf(target, prototype) {},      // Object.setPrototypeOf
  isExtensible(target) {},                   // Object.isExtensible
  preventExtensions(target) {},              // Object.preventExtensions
  getOwnPropertyDescriptor(target, prop) {}, // Object.getOwnPropertyDescriptor
  ownKeys(target) {},                        // Object.keys / Reflect.ownKeys
  defineProperty(target, prop, descriptor) {}// Object.defineProperty
};
```

---

## <div dir="rtl">سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **Proxy** | **Object.defineProperty** | **getter/setter** | **Immutable** |
|-----------|----------|--------------------------|-------------------|---------------|
| **Dynamic properties** | ✅ | ❌ | ❌ | ✅ |
| **New property auto-detect** | ✅ | ❌ | ❌ | ✅ |
| **Performance** | ⚠️ (overhead) | ⚡ | ⚡ | ⚡ |
| **Browser support** | ✅ (IE نه) | ✅ | ✅ | ✅ |
| **Reactive system** | ✅ | ✅ (Vue 2) | ✅ | ❌ |

</div>

<div dir="rtl">

### قانون Senior

۱. **Proxy**: برای reactive system پیشرفته (Vue 3)، validation framework، و logging/debugging.
۲. **Object.defineProperty**: برای getter/setter ساده و سازگاری با مرورگرهای قدیمی.
۳. **getter/setter**: برای computed properties ساده بدون نیاز به intercept.
۴. **هشدار**: Proxy قابلیت polyfill ندارد — برای IE11 غیرممکن است.

</div>

```javascript
// x Proxy برای validation — واقعی در production
function createValidated(schema) {
  return new Proxy({}, {
    set(target, prop, value) {
      const rule = schema[prop];
      if (rule && !rule.validate(value)) {
        throw new Error(`Invalid value for ${prop}: ${rule.message}`);
      }
      return Reflect.set(target, prop, value);
    }
  });
}

const user = createValidated({
  age:    { validate: v => v >= 0 && v <= 150, message: "0-150" },
  email:  { validate: v => v.includes("@"),    message: "invalid email" }
});

user.age = 25;     // ✅
user.age = -1;     // ❌ Error: Invalid value for age: 0-150
user.email = "a@b"; // ✅
```

---

## <div dir="rtl">سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

### سناریو: پیاده‌سازی Reactive System مثل Vue 3

Vue 3 از `Proxy` برای reactive state استفاده می‌کند. چالش واقعی: تشخیص تغییرات nested objects و جلوگیری از infinite loop.

</div>

```javascript
// x Reactive system ساده با Proxy
function reactive(obj) {
  return new Proxy(obj, {
    get(target, prop, receiver) {
      const value = Reflect.get(target, prop, receiver);
      if (typeof value === "object" && value !== null) {
        return reactive(value); // nested objects هم reactive
      }
      return value;
    },
    set(target, prop, value, receiver) {
      const oldValue = target[prop];
      const result = Reflect.set(target, prop, value, receiver);
      if (oldValue !== value) {
        console.log(`changed: ${prop}`);
        triggerEffects(prop); // dependency通知
      }
      return result;
    }
  });
}

// x واقعی: Vue 3 effect system
let activeEffect = null;
const targetMap = new WeakMap();

function track(target, prop) {
  if (!activeEffect) return;
  let depsMap = targetMap.get(target);
  if (!depsMap) targetMap.set(target, (depsMap = new Map()));
  let deps = depsMap.get(prop);
  if (!deps) depsMap.set(prop, (deps = new Set()));
  deps.add(activeEffect);
}

function trigger(target, prop) {
  const depsMap = targetMap.get(target);
  const deps = depsMap?.get(prop);
  deps?.forEach((effect) => effect());
}
```

<div dir="rtl">

### سناریو: proxy همراه Proxy detection

مشکل واقعی: بعضی کتابخانه‌ها بررسی می‌کنند آیا object Proxy است:

</div>

```javascript
// x تشخیص proxy
console.log(Proxy.prototype); // undefined — قابل تشخیص نیست

// x راه‌حل: symbol marker
const IS_PROXY = Symbol("isProxy");
function createProxy(target) {
  return new Proxy(target, {
    get(t, p, r) {
      if (p === IS_PROXY) return true;
      return Reflect.get(t, p, r);
    }
  });
}
```

---

## <div dir="rtl">سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

اگر Proxy در دسترس نبود (IE11، bundle size محدود):

</div>

<div dir="ltr">

| **روش** | **Proxy** | **defineProperty** | **Immutable.js** | **Zustand (immer)** |
|---------|----------|-------------------|-----------------|-------------------|
| **Nested reactive** | ✅ | ❌ (دستی) | ✅ (structural sharing) | ✅ |
| **Performance** | ⚠️ | ⚡ | ⚡ | ⚡ |
| **Polyfill** | ❌ | ✅ | ✅ | ✅ |
| **Bundle size** | 0 | 0 | ~15KB | ~3KB |

</div>

```javascript
// x جایگزین Proxy با defineProperty — Vue 2 style
function observe(obj) {
  Object.keys(obj).forEach((key) => {
    let value = obj[key];
    Object.defineProperty(obj, key, {
      get() { return value; },
      set(newValue) {
        value = newValue;
        console.log(`${key} changed to ${newValue}`);
        // dependency notification
      }
    });
  });
}

// x جایگزین با Immer (برای state updates)
import { produce } from "immer";
const next = produce(draft => {
  draft.user.name = "new name";
}, currentState);
```

<div dir="rtl">

> **نهایی:** Proxy ابزار قدرتمندی برای intercept کردن عملیات object است — پایه reactive systems مثل Vue 3. محدودیت: غیرقابل polyfill و overhead عملکردی. اگر نیاز به سازگاری با IE11 داری، از defineProperty (Vue 2 style) استفاده کن. برای state management در React، Immer بهترین جایگزین است.

</div>