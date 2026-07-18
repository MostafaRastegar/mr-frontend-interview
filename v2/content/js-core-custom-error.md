<div dir="rtl">

شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

لطفاً سوال "Custom Error (extends Error) و اهمیت `captureStackTrace` در V8" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

Error یک کلاس built-in است که stack trace و message را ذخیره می‌کند. Custom Error کلاسی است که از Error extends می‌کند تا type-specific error handling ممکن شود.

`Error.captureStackTrace(target, constructor)` — یک V8 API که stack trace را روی target object می‌نویسد و فریم‌های بالاتر از constructor را حذف می‌کند (clean stack). اگر از آن استفاده نکنی، constructor خود خطا در stack ظاهر می‌شود.

</div>

```javascript
// x Custom Error صحیح
class AppError extends Error {
  constructor(message, statusCode = 500, context = {}) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.context = context;
    this.timestamp = new Date().toISOString();

    // x V8-specific: constructor را از stack حذف می‌کند
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor);
    }
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`, 404, { resource, id });
  }
}

class ValidationError extends AppError {
  constructor(fields) {
    super('Validation failed', 400, { fields });
    this.fields = fields;
  }
}

// x استفاده
try {
  throw new NotFoundError('User', 42);
} catch (err) {
  console.log(err.name);       // "NotFoundError"
  console.log(err.statusCode); // 404
  console.log(err.context);    // { resource: 'User', id: 42 }
  console.log(err.stack);      // clean — constructor فریم ندارد
}
```

<div dir="rtl">

بدون `captureStackTrace`، stack شامل `new AppError` → `new NotFoundError` → your code می‌شود — نویز. با `captureStackTrace(this, this.constructor)` خط اول stack مستقیماً callsite است.

</div>

---

## <div dir="rtl">سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

| رویکرد | مزیت | محدودیت |
|--------|------|---------|
| `class extends Error` | type checking با `instanceof`, context اضافه | نیاز به `captureStackTrace` در V8 |
| `{ message, code }` object | ساده, بدون class | `instanceof` کار نمی‌کند, stack trace ضعیف |
| `throw new Error(msg)` | سریعترین | نمی‌توانی type-specific catch کنی |
| Third-party (verror, pino) | structured logging, cause chain | وابستگی |

</div>

```javascript
// Error cause chain (ES2022)
class DatabaseError extends Error {
  constructor(message, options) {
    super(message, options); // options.cause
    this.name = 'DatabaseError';
  }
}

try {
  throw new DatabaseError('Connection failed', {
    cause: new Error('ECONNREFUSED: 127.0.0.1:5432')
  });
} catch (err) {
  console.log(err.cause.message); // "ECONNREFUSED: 127.0.0.1:5432"
}
```

---

## <div dir="rtl">سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

**سناریو:** ۵ تیم مختلف هرکدام ماژول خودشان را توسعه می‌دهند. هر ماژول Errorهای مختلفی throw می‌کند. در middleware مرکزی (Error Boundary در React یا Express middleware) باید همه را catch کنی و response مناسب بدهی.

**چالش:** بدون Custom Error classes، هر تیم از `throw new Error('msg')` استفاده می‌کند — نمی‌توانی type را تشخیص دهی و status code یا پیام مناسب بدهی.

**راه‌حل:** Error hierarchy متمرکز + error serializer.

</div>

```javascript
// x Error hierarchy سازمان‌یافته
class DomainError extends AppError { /* generic domain */ }
class InfrastructureError extends AppError { /* DB, network */ }

// Express middleware
function errorHandler(err, req, res, next) {
  // Serialize to safe response
  const payload = {
    error: err.name,
    message: err.statusCode < 500
      ? err.message
      : 'Internal Server Error',
    ...(process.env.NODE_ENV === 'development' && {
      stack: err.stack,
      context: err.context,
    }),
  };

  // x Log کامل
  logger.error({
    err,
    requestId: req.id,
    userId: req.user?.id,
    path: req.path,
  });

  res.status(err.statusCode || 500).json(payload);
}

// React Error Boundary
class AppErrorBoundary extends React.Component {
  componentDidCatch(error, info) {
    if (error instanceof NotFoundError) {
      this.setState({ fallback: <NotFoundPage /> });
    } else if (error instanceof ValidationError) {
      this.setState({ fallback: <ValidationErrorUI fields={error.fields} /> });
    } else {
      reportToSentry(error, info);
      this.setState({ fallback: <ServerError /> });
    }
  }
}
```

<div dir="rtl">

**نکته Production:** `captureStackTrace` در V8 هزینه دارد (walk کردن call stack). در hot pathها (مثلاً validation که میلیون‌ها بار throw می‌شود) از `Error` ساده بدون stack استفاده کن — یا از `throw` اصلاً استفاده نکن و error return pattern بزن.

</div>

---

## <div dir="rtl">سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

بدون Error classes (ES5, non-V8 like Hermes):

</div>

| راهکار | توضیح | محدودیت |
|--------|-------|---------|
| Error objects دستی | `{ name, message, stack }` | `instanceof` کار نمی‌کند, prototype مفقود |
| `Object.create(Error.prototype)` | شبیه‌سازی ارث‌بری | verbose, `stack` property ممکن است نباشد |
| Result type (Rust style) | `{ ok, val }` / `{ err, msg }` | بدون throw, TypeScript discriminated union | 
| Either monad | `left = error, right = value` | کتابخانه (fp-ts), learning curve |

<div dir="rtl">

**مقایسه Next.js vs Express.js:** در Next.js Server Actions، Errorها در مرز client/server serialized می‌شوند — `instanceof` در سمت کلاینت کار نمی‌کند. باید از `error.name` یا `error.digest` استفاده کنی. در Express.js، `instanceof` در middleware مرکزی به راحتی کار می‌کند.

</div>

<div dir="rtl">

> **نهایی:** مهم‌ترین نکته Senior: `instanceof` با Custom Error classes در microservices/jest cross-realm (مثل iframe, VM context) خراب می‌شود — هر realm Error.prototype متفاوتی دارد. راه‌حل: از `error.name` یا `Symbol.for` برای cross-realm type checking استفاده کن. `captureStackTrace` را فراموش نکن — stack تمیز نشان‌دهنده‌ی professionalism است.

</div>
