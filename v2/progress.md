# Progress — AI Content Production

> هر ردیف `[ ]` یک مبحث تولیدنشده است. هر ردیف `[x]` یک مبحث تولیدشده.
> ترتیب تولید: اولویت ⭐⭐⭐ (فهرست ۱ → ۲ → ۳)، سپس ⭐⭐، سپس ⭐.

---

## فهرست ۱: JavaScript/TypeScript Core

### ⭐⭐⭐ (ضروری)
- [x] 1. Event Loop معماری (Call Stack, Microtask Queue vs Macrotask Queue)
- [x] 2. Closure و کاربرد آن در Factory Functions و Module Pattern
- [x] 3. تفاوت عمیق == vs === (الگوریتم Abstract Equality Comparison)
- [x] 4. Hoisting در var, let, const و function declarations (TDZ)
- [x] 5. this در ۴ حالت مختلف (Global, Function, Arrow Function, کلاس‌ها)
- [x] 6. call, apply, bind (و پیاده‌سازی bind ساده با Closure)
- [x] 7. Prototype و Prototypal Inheritance
- [x] 8. Array متدهای پیشرفته (map, filter, reduce, some, every, flatMap)

### ⭐⭐ (مهم)
- [x] 9. Symbol و کاربرد آن در ایجاد کلیدهای خصوصی
- [x] 10. WeakMap و WeakSet (تفاوت با Map/Set و مدیریت حافظه)
- [x] 11. Generator Functions و Iterators
- [x] 12. Proxy و Reflect (کاربرد در Validation, Trapping, Reactive Programming)
- [x] 13. Debouncing و Throttling (پیاده‌سازی دستی با Closure)
- [x] 14. Deep Copy vs Shallow Copy (structuredClone و JSON.stringify)
- [x] 15. Promise و Async/Await (Promise.all, race, allSettled)
- [x] 16. Custom Error (extends Error) و captureStackTrace در V8

### ⭐ (پیشرفته)
- [x] 17. Coercion (تبدیل ضمنی نوع‌ها) و ToPrimitive و toStringTag
- [x] 18. Tail Call Optimization در V8 و محدودیت‌های آن در Node.js
- [x] 19. SharedArrayBuffer و Atomics برای محاسبات موازی در Web Workers
- [x] 20. FinalizationRegistry برای مدیریت حافظه و پاک‌سازی منابع
- [x] 21. Composition vs Inheritance در سطح کلاس‌های JS
- [x] 22. Currying و Partial Application (و تفاوت آن‌ها)
- [x] 23. Memoization در سطح توابع خالص (Pure Functions) و WeakMap برای Cache
- [x] 24. Scheduler و postTask برای اولویت‌بندی تسک‌ها در مرورگر
- [x] 25. Error Boundary در سطح Vanilla JS

---
---

## فهرست ۲: React.js + Next.js

### ⭐⭐⭐ (ضروری)
- [x] 1. Reconciliation و الگوریتم Diffing (اهمیت کلید پایدار)
- [x] 2. useState و useReducer (تفاوت در مدیریت State پیچیده)
- [x] 3. useEffect (دپندنسی‌ها، clean-up function، useLayoutEffect)
- [x] 4. useRef (۳ کاربرد اصلی: mutable, DOM, جلوگیری از re-render)
- [x] 5. Context API (محدودیت‌ها در re-rendering و بهینه‌سازی)
- [x] 6. HOC vs Render Props vs Hooks (تاریخچه و مزایای Hooks)
- [x] 7. React.memo و PureComponent (shallow compare)
- [x] 8. useMemo vs useCallback (تفاوت ماهوی و تأثیر بر GC)
- [x] 9. Lazy Loading و Suspense (تأثیر بر FCP و LCP)
- [x] 10. Controlled vs Uncontrolled Components

### ⭐⭐ (مهم)
- [x] 11. Server Components vs Client Components (use client و use server)
- [x] 12. Data Fetching در Next.js 14+ (fetch, cache, revalidate)
- [x] 13. Middleware در Next.js (مقایسه با Express.js middleware)
- [x] 14. App Router vs Pages Router (تفاوت و استراتژی مهاجرت)
- [x] 15. ISR (Incremental Static Regeneration) (Fallback و کش CDN)
- [x] 16. Dynamic Routes و Catch-all Routes (تأثیر بر getStaticPaths)
- [x] 17. API Routes و Route Handlers (اعتبارسنجی با Zod)
- [x] 18. next/head و Metadata API (تأثیر بر SEO و Open Graph)
- [x] 19. next/image (بهینه‌سازی تصاویر، sizes, priority و LCP)
- [x] 20. next/script (afterInteractive vs lazyOnload)

### ⭐ (پیشرفته)
- [x] 21. React Fiber معماری (تقسیم کار و Concurrent Mode)
- [x] 22. Concurrent Rendering و Transitions (useTransition, useDeferredValue)
- [x] 23. React Server Components Payload (انتقال داده سرور به کلاینت)
- [x] 24. Module Federation در Next.js (Micro-frontend)
- [x] 25. Turbopack vs Webpack (تفاوت معماری و زمان بیلد)
- [x] 26. Streaming SSR و Suspense (رندر تدریجی HTML و TTFB)
- [x] 27. Edge Runtime vs Node.js Runtime
- [x] 28. Cache Invalidation استراتژی‌ها (بر اساس tags و paths)

---

## فهرست ۳: معماری، مهندسی نرم‌افزار، تست و پرفورمنس

### ⭐⭐⭐ (ضروری)
- [x] 1. State Management Strategy (Redux, Zustand, Context, React Query)
- [x] 2. Code Splitting استراتژی‌ها (Route-based, Component-based)
- [x] 3. Performance Metrics (LCP, FID, CLS, TTI, TBT)
- [x] 4. Lighthouse و Web Vitals (اندازه‌گیری در Production)
- [x] 5. Testing (Unit, Integration, E2E)
- [x] 6. Mocking در تست‌ها (ماژول‌ها، API calls، توابع تاریخ/زمان)
- [x] 7. Error Handling استراتژی (Error Boundaries, Server Components, Sentry)
- [x] 8. Logging و Monitoring (console.log vs APM در Production)
- [x] 9. Security (XSS, CSRF, SQL Injection, next/headers)
- [x] 10. Environment Variables (NEXT_PUBLIC_* vs سمت سرور)

### ⭐⭐ (مهم)
- [x] 11. Monorepo (Turborepo, Nx) و مدیریت اشتراک‌گذاری کد
- [x] 12. Micro-frontend استراتژی‌ها (Module Federation, iframe, Web Components)
- [x] 13. Design Patterns در React (Container/Presentational, Compound Components)
- [x] 14. SOLID اصول در Frontend (Open/Closed, Liskov Substitution)
- [x] 15. Clean Code و Code Review (معیارهای قبولی PR)
- [x] 16. Refactoring استراتژی‌ها (تقسیم کامپوننت ۵۰۰ خطی)
- [x] 17. Bundle Analysis (@next/bundle-analyzer)
- [x] 18. Tree Shaking (عملکرد و محدودیت‌ها با Side Effects)
- [x] 19. Lazy Loading در سطح Routing (prefetching)
- [x] 20. PWA (Service Worker, Workbox, کشینگ آفلاین)

### ⭐ (پیشرفته)
- [x] 21. Scaling (CDN, Edge Computing, Database Replication)
- [x] 22. CI/CD Pipeline (GitHub Actions, GitLab CI, Vercel/AWS)
- [x] 23. Feature Flags (LaunchDarkly یا خودمانی)
- [x] 24. A/B Testing (پیاده‌سازی در سطح سرور با Next.js)
- [x] 25. API Design (RESTful vs GraphQL vs tRPC)
- [x] 26. Caching Strategy (CDN, Redis, Browser Cache)
- [x] 27. Database Interaction (Prisma/Drizzle, Connection Pool)
- [x] 28. WebSocket و Real-time (Socket.io با Next.js)
- [x] 29. Accessibility (a11y) (aria-*, role, axe-core)
- [x] 30. Internationalization (i18n) (next-intl, react-i18next)