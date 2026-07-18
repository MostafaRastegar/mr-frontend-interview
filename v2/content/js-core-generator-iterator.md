<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Generator Functions و Iterators (و نحوه‌ی پیاده‌سازی یک Observable با آن‌ها)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Generator Function** (با `function*`) تابعی است که می‌تواند اجرای خود را متوقف کند و بعداً از همان نقطه ادامه دهد. هر بار که `yield` می‌کند، یک مقدار به caller برمی‌گرداند و منتظر می‌ماند تا دوباره `next()` صدا شود. نتیجه‌ی یک Generator یک **Generator Object** است که **Iterator Protocol** را پیاده‌سازی می‌کند.

‏**Iterator** یک object است که متد `next()` دارد و یک object با شکل `{ value, done }` برمی‌گرداند.

</div>

```javascript
// Generator Function
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// x Generator object همزمان Iterable و Iterator
console.log(gen[Symbol.iterator]() === gen); // true

// x Iterator Protocol ساده
const iterator = {
  current: 0,
  next() {
    if (this.current < 3) {
      return { value: this.current++, done: false };
    }
    return { value: undefined, done: true };
  },
  // x برای اینکه در for..of کار کند
  [Symbol.iterator]() { return this; }
};

for (const val of iterator) {
  console.log(val); // 0, 1, 2
}
```

<div dir="rtl">

‏### خصوصیات فنی در سطح V8/موتور

‏- **Lazy Evaluation**: Generatorها مقادیر را یک‌به‌یک تولید می‌کنند — کل sequence در حافظه ساخته نمی‌شود.
‏- **State Machine**: هر Generator یک state machine داخلی دارد که موقعیت فعلی (yield point) را ذخیره می‌کند.
‏- **Two-way Communication**: با `gen.next(value)` می‌توانی مقدار به Generator بفرستی. با `gen.throw(error)` می‌توانی error پرتاب کنی داخل Generator.
‏- **yield* Delegate**: با `yield*` می‌توانی به Generator دیگر delegation کنی.

</div>

```javascript
// Two-way communication
function* twoWay() {
  const x = yield "give me x"; // اینجا منتظر می‌ماند
  const y = yield "give me y";
  return x + y;
}

const g = twoWay();
console.log(g.next());     // { value: "give me x", done: false }
console.log(g.next(5));    // { value: "give me y", done: false } — x = 5
console.log(g.next(3));    // { value: 8, done: true } — y = 3, return 8

// yield* delegation
function* gen1() {
  yield 1;
  yield 2;
}
function* gen2() {
  yield* gen1(); // delegate به gen1
  yield 3;
}
console.log([...gen2()]); // [1, 2, 3]
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="ltr">

| **سناریو** | **Generator** | **Array** | **Async/Await** | **RxJS/Observable** |
|-----------|--------------|-----------|-----------------|---------------------|
| **Lazy evaluation** | ✅ | ❌ | ❌ | ✅ |
| **مقادیر بی‌نهایت** | ✅ | ❌ | ❌ | ✅ |
| **Async data** | ❌ (نیاز به wrapper) | ❌ | ✅ | ✅ |
| **Two-way comm** | ✅ | ❌ | ❌ | ❌ |
| **پیچیدگی** | متوسط | کم | کم | زیاد |
| **Performance** | ⚡ | ⚡ (برای dataset کوچک) | ⚡ | ⚠️ (overhead) |

</div>

<div dir="rtl">

‏### قانون Senior

‏۱. **Generator**: برای تولید داده‌های lazy یا بی‌نهایت (مثل pagination, Fibonacci, range) و پیاده‌سازی async flow control.
‏۲. **Array**: برای dataset محدود که نیاز به random access داری.
‏۳. **Async/Await**: برای async operations معمولی.
‏۴. **RxJS**: برای data streams پیچیده که نیاز به composition, debounce, retry دارند.

</div>

```javascript
// x Generator برای pagination (lazy)
function* paginate(items, pageSize) {
  for (let i = 0; i < items.length; i += pageSize) {
    yield items.slice(i, i + pageSize);
  }
}

const pages = paginate([1,2,3,4,5,6,7,8,9,10], 3);
for (const page of pages) {
  console.log(page); // [1,2,3], [4,5,6], [7,8,9], [10]
}

// x Generator برای Fibonacci (بی‌نهایت)
function* fibonacci() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const fib = fibonacci();
console.log(fib.next().value); // 0
console.log(fib.next().value); // 1
console.log(fib.next().value); // 1
console.log(fib.next().value); // 2
console.log(fib.next().value); // 3
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏### سناریو: پیاده‌سازی Observable با Generator

‏در یک پروژه Dashboard با داده‌های real-time، نیاز به یک Observable ساده داشتیم که از WebSocket events را به صورت lazy پردازش کند. نمی‌خواستیم RxJS را اضافه کنیم (حجم bundle). از Generator استفاده کردیم:

</div>

```javascript
// x Observable ساده با Generator
function createObservable(subscribeFn) {
  return {
    subscribe(observer) {
      const generator = createGenerator(observer);
      const unsubscribe = subscribeFn(generator);
      return { unsubscribe };
    }
  };
}

function* createGenerator(observer) {
  try {
    while (true) {
      const value = yield;
      observer.next(value);
    }
  } catch (error) {
    observer.error(error);
  } finally {
    observer.complete();
  }
}

// x استفاده
const ws$ = createObservable((generator) => {
  const ws = new WebSocket("wss://api.example.com/stream");
  ws.onmessage = (event) => generator.next(JSON.parse(event.data));
  ws.onerror = (error) => generator.throw(error);
  ws.onclose = () => generator.return();
  return () => ws.close();
});

const subscription = ws$.subscribe({
  next: (data) => console.log("data:", data),
  error: (err) => console.error("error:", err),
  complete: () => console.log("stream closed")
});
```

<div dir="rtl">

‏### سناریوی واقعی: Async Flow Control با Generator (قبل از Async/Await)

‏در پروژه‌های قدیمی، قبل از ES2017، از Generatorها برای مدیریت async flow استفاده می‌کردیم:

</div>

```javascript
// x Async flow control با Generator — شبیه async/await
function asyncRunner(generatorFn) {
  return function (...args) {
    const gen = generatorFn(...args);

    function step(res) {
      const result = gen.next(res);
      if (result.done) return Promise.resolve(result.value);
      return Promise.resolve(result.value).then(
        (val) => step(val),
        (err) => gen.throw(err)
      );
    }

    return step();
  };
}

// x استفاده
const fetchUser = asyncRunner(function* (id) {
  const response = yield fetch(`/api/users/${id}`);
  const user = yield response.json();
  return user;
});

fetchUser(1).then((user) => console.log(user));
```

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزینه</div>

<div dir="rtl">

‏اگر Generator در دسترس نبود یا نیاز به قابلیت‌های پیشرفته‌تر داشتی:

</div>

<div dir="ltr">

| **روش** | **Generator** | **RxJS** | **Async Iterator** | **Custom Iterator** |
|---------|--------------|----------|-------------------|-------------------|
| **Lazy** | ✅ | ✅ | ✅ | ✅ |
| **Async** | ❌ | ✅ | ✅ | ❌ |
| **Composition** | محدود | ✅ (operators) | محدود | محدود |
| **Bundle size** | 0 (native) | ~50KB min | 0 (native) | متغیر |
| **Backpressure** | دستی | ✅ | دستی | دستی |

</div>

```javascript
// x جایگزین Generator با RxJS (برای streams پیچیده)
import { fromEvent, map, filter, debounceTime } from 'rxjs';

const input$ = fromEvent(input, 'input');
input$.pipe(
  map((e) => e.target.value),
  filter((val) => val.length > 2),
  debounceTime(300)
).subscribe((val) => console.log(val));

// x جایگزین Generator با Async Iterator (ES2018)
async function* asyncPaginate(url, limit = 10) {
  let page = 1;
  while (true) {
    const response = await fetch(`${url}?page=${page}&limit=${limit}`);
    const data = await response.json();
    if (data.length === 0) return;
    yield data;
    page++;
  }
}

for await (const page of asyncPaginate('/api/users')) {
  console.log(page);
  if (page.some(user => user.role === 'admin')) break;
}

// x جایگزین Generator با custom iterator
function createRange(start, end) {
  let current = start;
  return {
    next() {
      if (current <= end) {
        return { value: current++, done: false };
      }
      return { value: undefined, done: true };
    },
    [Symbol.iterator]() { return this; }
  };
}

for (const num of createRange(1, 5)) {
  console.log(num); // 1, 2, 3, 4, 5
}
```

<div dir="rtl">

‏> **نهایی:** Generator برای lazy evaluation و تولید داده‌های بی‌نهایت عالی است. محدودیت: async support ندارد (مگر با Async Iterator که ES2018 است). اگر نیاز به data streams پیچیده (retry, debounce, combine) داری، RxJS انتخاب بهتری است. Async Iterator جایگزین native برای async lazy data است.

</div>