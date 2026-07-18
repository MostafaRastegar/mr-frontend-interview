<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Tail Call Optimization در V8 و محدودیت‌های آن در Node.js" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Tail Call:** وقتی آخرین عملیات یک تابع، فراخوانی تابع دیگری است و نتیجه‌ی آن مستقیماً برگردانده می‌شود.

‏**TCO (Tail Call Optimization):** به جای push stack frame جدید، current frame را با frame جدید جایگزین می‌کند — حافظه O(1) به جای O(n).

‏**PTC (Proper Tail Call):** استاندارد ES2015 — پیاده‌سازی اجباری در specification. اما V8 فقط در strict mode و با flagهای خاص پشتیبانی می‌کند.

</div>

```javascript
'use strict';

// Tail call position ✓
function factorial(n, acc = 1) {
  if (n <= 1) return acc;
  return factorial(n - 1, n * acc); // tail call
}

// NOT tail call ✗
function factorialBad(n) {
  if (n <= 1) return 1;
  return n * factorialBad(n - 1); // بعد از بازگشت، ضرب انجام می‌شود
}

// x Tail call در V8 strict mode
function sum(arr, i = 0, acc = 0) {
  if (i >= arr.length) return acc;
  return sum(arr, i + 1, acc + arr[i]); // tail
}

// x مقایسه stack
console.log(factorialBad(100000)); // RangeError: Maximum call stack size exceeded
console.log(factorial(100000));    // V8 strict mode → Infinity, non-strict → stack overflow
```

<div dir="rtl">

‏واقعیت: V8 TCO را در strict mode پیاده‌سازی کرد اما بعداً آن را **غیرفعال** کرد (فقط در edge/embedded کار می‌کند). Node.js و Chrome فعلاً TCO را پشتیبانی نمی‌کنند — مگر با `--harmony-tailcalls` (experimental, حذف شده).

</div>

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏| رویکرد | مزیت | محدودیت |
‏|--------|------|---------|
‏| Tail recursion + TCO | O(1) stack | V8 پشتیبانی نمی‌کند (عملاً) |
‏| Iterative (loop) | O(1) stack, predictable | verbosity برای مسائل recursive |
‏| Trampoline | شبیه‌سازی tail call در ES5 | overhead تابع, پیچیدگی |
‏| Generator-based recursion | lazy evaluation | نرخ سربار بالا |

</div>

```javascript
// x Trampoline — جایگزین TCO در runtime
function trampoline(fn) {
  return function (...args) {
    let result = fn(...args);
    while (typeof result === 'function') {
      result = result();
    }
    return result;
  };
}

const safeFactorial = trampoline(
  (n, acc = 1) => n <= 1 ? acc : () => safeFactorial(n - 1, n * acc)
);

console.log(safeFactorial(100000)); // Infinity (بدون stack overflow)
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏**سناریو:** یک سرویس پردازش استعلام از دیتابیس گراف (graph) با عمق نامحدود — recursive graph traversal بدون TCO ممکن است stack را بشکند.

‏**چالش:** V8 stack size محدود (~۱۲۰۰۰ frame). اگر گراف ۲۰۰۰۰ نود depth داشته باشد، recursive implementation crash می‌کند.

‏**راه‌حل:** Iterative implementation با explicit stack + مدیریت context.

</div>

```javascript
// x Graph traversal — iterative با explicit stack
function findPath(graph, start, target) {
  const stack = [{ node: start, path: [start], visited: new Set() }];
  stack[0].visited.add(start);

  while (stack.length > 0) {
    const { node, path, visited } = stack.pop();

    if (node === target) return path;

    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        const newVisited = new Set(visited);
        newVisited.add(neighbor);
        stack.push({
          node: neighbor,
          path: [...path, neighbor],
          visited: newVisited,
        });
      }
    }
  }

  return null; // not found
}

// Deep object transform — iterative
function deepMap(obj, fn) {
  const stack = [{ source: obj, target: Array.isArray(obj) ? [] : {} }];
  const root = stack[0].target;

  while (stack.length > 0) {
    const { source, target } = stack.pop();

    for (const key of Object.keys(source)) {
      const val = source[key];
      if (val !== null && typeof val === 'object') {
        const newTarget = Array.isArray(val) ? [] : {};
        target[key] = newTarget;
        stack.push({ source: val, target: newTarget });
      } else {
        target[key] = fn(val, key);
      }
    }
  }

  return root;
}
```

<div dir="rtl">

‏**نکته Production:** همیشه عمق ورودی را validation کن — اگر recursive algorithm داری و عمق می‌تواند > ۱۰۰۰ باشد، به iterative refactor کن. Tail call optimization promise است، reality نیست.

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏بدون TCO (V8, SpiderMonkey):

</div>

| روش | کاربرد | توضیح |
|-----|--------|-------|
| Iterative loop | همیشه | هیچ recursion overheadی ندارد |
| `while` + stack | tree/graph traversal | explicit, predictable |
| `p-limit` + queue | async recursive (API calls) | depth limit + concurrency |
| Web Workers | عملیات سنگین | محاسبات در ترد جدا |
| WASM | recursion-heavy algorithms | TCO در WASM ممکن است (LLVM backend) |

<div dir="rtl">

‏**مقایسه Next.js vs Express.js:** هر دو روی Node.js/V8 اجرا می‌شوند — TCO در هیچکدام کار نمی‌کند. اما Next.js API routes معمولاً عمق recursion محدودی دارند (request → handler → response) در حالی که Express.js middleware chain می‌تواند عمیق شود (middleware تو در تو). در Express.js اگر middlewareهای سفارشی recursive باشند (مثلاً waterfall authentication), iterative approach امن‌تر است.

</div>

<div dir="rtl">

‏> **نهایی:** TCO در تئوری عالی است اما در V8 واقعیت ندارد. همیشه iteration را به recursion ترجیح بده وقتی عمق نامشخص است. اگر recursion ضروری است (tree traversal, graph algorithms), از explicit stack با `while` استفاده کن. Trampoline یک جایگزین آکادمیک است اما در production `while` ساده بهتر است — خوانا، سریع، predictable.

</div>
