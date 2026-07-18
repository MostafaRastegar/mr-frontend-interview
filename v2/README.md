> 📊 **Progress Tracking**: See `progress.md` for production status of all topics. Generated articles go in `content/INDEX.md`.

یک مهندس Senior باید در **سه اکوسیستم مجزا** به صورت هم‌زمان مسلط باشد:
1. **هسته سخت (Vanilla JS/TS):** جایی که اگر بلد نباشی، حتی با بهترین ابزارها هم آبرویت می‌رود (مثل مدیریت `this`، Closure، یا Event Loop).
2. **ابزارهای اصلی (React + Next.js):** جایی که باید نشان دهی چگونه از قابلیت‌های جدید (مثل Server Components) درست استفاده می‌کنی.
3. **مهندسی نرم‌افزار و معماری (Design Patterns, Testing, Security):** جایی که سطح Senior بودن را از Mid-level جدا می‌کند.

برای هر کدام از این ۳ محور، یک **فهرست بلند (۲۵-۳۰ تایی)** با همان فرمت تیتروار ولی با سطح‌بندی اهمیت (ضروری، مهم، پیشرفته) تهیه کرده‌ام. 

---

## فهرست شماره ۱: JavaScript/TypeScript Core (بخش آبروریزان!)
*این بخش را اگر بلد نباشی، با وجود ۱۰ سال سابقه، به عنوان `Mid-level` خطاب می‌شوی.*

**سطح ⭐⭐⭐ (ضروری و حتماً پرسیده می‌شود):**
1. [📄](content/js-core-event-loop.md) `Event Loop` معماری (Call Stack, Microtask Queue vs Macrotask Queue)
2. [📄](content/js-core-closure.md) `Closure` و کاربرد آن در Factory Functions و Module Pattern
3. [📄](content/js-core-equality-comparison.md) تفاوت عمیق `==` vs `===` (الگوریتم Abstract Equality Comparison)
4. [📄](content/js-core-hoisting.md) `Hoisting` در `var`، `let`، `const` و `function declarations` (TDZ)
5. [📄](content/js-core-this.md) `this` در ۴ حالت مختلف (Global، Function، Arrow Function، و در کلاس‌ها)
6. [📄](content/js-core-call-apply-bind.md) `call`، `apply`، و `bind` (و پیاده‌سازی یک `bind` ساده با Closure)
7. [📄](content/js-core-prototype.md) `Prototype` و `Prototypal Inheritance`
8. [📄](content/js-core-array-methods.md) `Array` متدهای پیشرفته (`map`, `filter`, `reduce`, `some`, `every`, `flatMap`)

**سطح ⭐⭐ (مهم و نشان‌دهنده‌ی تسلط بالا):**
9. [📄](content/js-core-symbol.md) `Symbol` و کاربرد آن در ایجاد کلیدهای خصوصی
10. [📄](content/js-core-weakmap-weakset.md) `WeakMap` و `WeakSet` (تفاوت با Map/Set و مدیریت حافظه)
11. [📄](content/js-core-generator-iterator.md) `Generator Functions` و `Iterators`
12. [📄](content/js-core-proxy-reflect.md) `Proxy` و `Reflect` (کاربرد در Validation, Trapping, Reactive Programming)
13. [📄](content/js-core-debounce-throttle.md) `Debouncing` و `Throttling` (پیاده‌سازی دستی با Closure)
14. [📄](content/js-core-deep-shallow-copy.md) `Deep Copy` vs `Shallow Copy` (مشکل `structuredClone` و JSON.stringify)
15. [📄](content/js-core-promise-async-await.md) `Promise` و `Async/Await` (مدیریت خطا، `Promise.all`, `Promise.race`, `Promise.allSettled`)
16. [📄](content/js-core-custom-error.md) `Custom Error` (extends Error) و اهمیت `captureStackTrace` در V8

**سطح ⭐ (پیشرفته و تمایزدهنده برای Senior Staff):**
17. [📄](content/js-core-coercion.md) `Coercion` (تبدیل ضمنی نوع‌ها) و متدهای `ToPrimitive` و `toStringTag`
18. [📄](content/js-core-tail-call-optimization.md) `Tail Call Optimization` در V8 و محدودیت‌های آن در Node.js
19. [📄](content/js-core-sharedarraybuffer-atomics.md) `SharedArrayBuffer` و `Atomics` برای محاسبات موازی در Web Workers
20. [📄](content/js-core-finalization-registry.md) `FinalizationRegistry` برای مدیریت حافظه و پاک‌سازی منابع
21. [📄](content/js-core-composition-vs-inheritance.md) `Composition vs Inheritance` در سطح کلاس‌های JS
22. [📄](content/js-core-currying-partial.md) `Currying` و `Partial Application` (و تفاوت آن‌ها)
23. [📄](content/js-core-memoization.md) `Memoization` در سطح توابع خالص (Pure Functions) و WeakMap برای Cache
24. [📄](content/js-core-scheduler-posttask.md) `Scheduler` و `postTask` برای اولویت‌بندی تسک‌ها در مرورگر
25. [📄](content/js-core-error-boundary.md) `Error Boundary` در سطح Vanilla JS

---

## فهرست شماره ۲: React.js + Next.js (بخش ابزارها و فریم‌ورک)
*اینجا باید نشان دهی که فقط مصرف‌کننده نیستی و از جزئیات داخلی (Internal) مطلعی.*

**سطح ⭐⭐⭐ (ضروری و روزمره):**
1. [📄](content/react-reconciliation.md) `Reconciliation` و الگوریتم Diffing (اهمیت کلید پایدار)
2. [📄](content/react-usestate-usereducer.md) `useState` و `useReducer` (تفاوت در مدیریت State پیچیده)
3. [📄](content/react-useeffect.md) `useEffect` (دپندنسی‌ها، clean-up function، و `useLayoutEffect`)
4. [📄](content/react-useref.md) `useRef` (۳ کاربرد اصلی: mutable, DOM, جلوگیری از re-render)
5. [📄](content/react-context-api.md) `Context API` (محدودیت‌ها در re-rendering و بهینه‌سازی)
6. [📄](content/react-hoc-render-props-hooks.md) `HOC` vs `Render Props` vs `Hooks` (تاریخچه و مزایای Hooks)
7. [📄](content/react-memo-purecomponent.md) `React.memo` و `PureComponent` (shallow compare)
8. [📄](content/react-usememo-usecallback.md) `useMemo` vs `useCallback` (تفاوت ماهوی و تأثیر بر GC)
9. [📄](content/react-lazy-suspense.md) `Lazy Loading` و `Suspense` (تأثیر بر FCP و LCP)
10. [📄](content/react-controlled-uncontrolled.md) `Controlled vs Uncontrolled Components`

**سطح ⭐⭐ (مهم و نشان‌دهنده‌ی تسلط بر Next.js):**
11. [📄](content/react-server-client-components.md) `Server Components` vs `Client Components` (`'use client'` و `'use server'`)
12. [📄](content/react-data-fetching.md) `Data Fetching` در Next.js 14+ (`fetch` با `cache`, `next: { revalidate }`)
13. [📄](content/react-middleware.md) `Middleware` در Next.js (مقایسه با Express.js middleware)
14. [📄](content/react-app-router-vs-pages-router.md) `App Router` vs `Pages Router` (تفاوت و استراتژی مهاجرت)
15. [📄](content/react-isr.md) `ISR (Incremental Static Regeneration)` (مدیریت Fallback و کش CDN)
16. [📄](content/react-dynamic-routes.md) `Dynamic Routes` و `Catch-all Routes` (تأثیر بر `getStaticPaths`)
17. [📄](content/react-api-routes.md) `API Routes` و `Route Handlers` (اعتبارسنجی با Zod)
18. [📄](content/react-metadata-api.md) `next/head` و `Metadata API` (تأثیر بر SEO و Open Graph)
19. [📄](content/react-next-image.md) `next/image` (بهینه‌سازی تصاویر، `sizes`, `priority` و LCP)
20. [📄](content/react-next-script.md) `next/script` (مدیریت اسکریپت‌ها و استراتژی `afterInteractive` vs `lazyOnload`)

**سطح ⭐ (پیشرفته و معماری داخلی):**
21. [📄](content/react-fiber-architecture.md) `React Fiber` معماری (تقسیم کار و Concurrent Mode)
22. [📄](content/react-concurrent-rendering.md) `Concurrent Rendering` و `Transitions` (`useTransition`, `useDeferredValue`)
23. [📄](content/react-rsc-payload.md) `React Server Components Payload` (انتقال داده سرور به کلاینت)
24. [📄](content/react-module-federation.md) `Module Federation` در Next.js (Micro-frontend)
25. [📄](content/react-turbopack-vs-webpack.md) `Turbopack` vs `Webpack` (تفاوت معماری و زمان بیلد)
26. [📄](content/react-streaming-ssr.md) `Streaming SSR` و `Suspense` در Next.js (رندر تدریجی HTML و TTFB)
27. [📄](content/react-edge-vs-node-runtime.md) `Edge Runtime` vs `Node.js Runtime` (محدودیت‌ها و زمان استفاده)
28. [📄](content/react-cache-invalidation.md) `Cache Invalidation` استراتژی‌ها (بر اساس `tags` و `paths`)

---

## فهرست شماره ۳: معماری، مهندسی نرم‌افزار، تست و پرفورمنس (بخش تمایز Senior)
*اینجا سوالات به‌صورت سناریو محور پرسیده می‌شود و اگر فقط کد بلد باشی، زمین می‌خوری.*

**سطح ⭐⭐⭐ (ضروری و بالاتر از Mid-level):**
1. [📄](content/arch-state-management.md) `State Management Strategy` (کدام ابزار برای چه سناریویی؟)
2. [📄](content/arch-code-splitting.md) `Code Splitting` استراتژی‌ها (Route-based, Component-based)
3. [📄](content/arch-performance-metrics.md) `Performance Metrics` (LCP, FID, CLS, TTI, TBT)
4. [📄](content/arch-lighthouse-web-vitals.md) `Lighthouse` و `Web Vitals` (اندازه‌گیری در Production)
5. [📄](content/arch-testing-strategy.md) `Testing` (Unit با Jest/Vitest، Integration با RTL، E2E با Playwright)
6. [📄](content/arch-mocking.md) `Mocking` در تست‌ها (ماژول‌ها، API calls، توابع تاریخ/زمان)
7. [📄](content/arch-error-handling.md) `Error Handling` استراتژی (Error Boundaries, Server Components, Sentry)
8. [📄](content/arch-logging-monitoring.md) `Logging` و `Monitoring` (console.log vs APM در Production)
9. [📄](content/arch-security.md) `Security` (XSS, CSRF, SQL Injection, `next/headers`)
10. [📄](content/arch-env-variables.md) `Environment Variables` (`NEXT_PUBLIC_*` vs سمت سرور)

**سطح ⭐⭐ (مهم برای پروژه‌های بزرگ):**
11. [📄](content/arch-monorepo.md) `Monorepo` (Turborepo یا Nx) و مدیریت اشتراک‌گذاری کد
12. [📄](content/arch-micro-frontend.md) `Micro-frontend` استراتژی‌ها (Module Federation vs iframe vs Web Components)
13. [📄](content/arch-design-patterns.md) `Design Patterns` در React (Container/Presentational, Compound Components)
14. [📄](content/arch-solid-principles.md) `SOLID` اصول در Frontend (Open/Closed, Liskov Substitution)
15. [📄](content/arch-clean-code-review.md) `Clean Code` و `Code Review` (معیارهای قبولی PR)
16. [📄](content/arch-refactoring.md) `Refactoring` استراتژی‌ها (تقسیم کامپوننت ۵۰۰ خطی)
17. [📄](content/arch-bundle-analysis.md) `Bundle Analysis` (`@next/bundle-analyzer`)
18. [📄](content/arch-tree-shaking.md) `Tree Shaking` (عملکرد و محدودیت‌ها با Side Effects)
19. [📄](content/arch-lazy-loading-routing.md) `Lazy Loading` در سطح Routing (prefetching)
20. [📄](content/arch-pwa.md) `PWA` (Service Worker, Workbox, کشینگ آفلاین)

**سطح ⭐ (پیشرفته و معماری سیستمی):**
21. [📄](content/arch-scaling.md) `Scaling` (CDN, Edge Computing, Database Replication)
22. [📄](content/arch-cicd-pipeline.md) `CI/CD Pipeline` (GitHub Actions, GitLab CI, Vercel/AWS)
23. [📄](content/arch-feature-flags.md) `Feature Flags` (LaunchDarkly یا خودمانی)
24. [📄](content/arch-ab-testing.md) `A/B Testing` (پیاده‌سازی در سطح سرور با Next.js)
25. [📄](content/arch-api-design.md) `API Design` (RESTful vs GraphQL vs tRPC)
26. [📄](content/arch-caching-strategy.md) `Caching Strategy` (CDN, Redis, Browser Cache)
27. [📄](content/arch-database-interaction.md) `Database Interaction` در Next.js (Prisma/Drizzle, Connection Pool)
28. [📄](content/arch-websocket-realtime.md) `WebSocket` و `Real-time` (Socket.io با Next.js)
29. [📄](content/arch-accessibility.md) `Accessibility (a11y)` (`aria-*`, `role`, axe-core)
30. [📄](content/arch-internationalization.md) `Internationalization (i18n)` (`next-intl`, `react-i18next`)

---

## جمع‌بندی نهایی برای استراتژی مطالعه:

| اولویت | تعداد سوالات | زمان مطالعه پیشنهادی | نحوه‌ی مطالعه |
| :--- | :--- | :--- | :--- |
| **فوری (تا یک هفته آینده)** | ۱۵ سوال اول از فهرست ۱ (JS Core) + ۱۰ سوال اول از فهرست ۲ (React/Next) | روزی ۳-۴ ساعت | دقیقاً با تمپلیت ۴ لایه‌ای که ساختی. حتماً برای هرکدام یک سناریوی واقعی از پروژه‌های قبلی‌ات پیدا کن. |
| **مهم (تا دو هفته آینده)** | باقی‌مانده‌ی فهرست ۱ و ۲ | روزی ۲ ساعت | تمرکز روی کد snippets و پیاده‌سازی دستی (مثل نوشتن یک `useThrottle` یا `Promise.all` دستی) |
| **تمایز (تا زمان مصاحبه)** | فهرست ۳ (معماری و پرفورمنس) به‌صورت انتخابی (۲۰ تا از ۳۰ تایی که به پروژه‌ات مربوط‌تر است) | روزی ۱ ساعت | مطالعه‌ی کیس‌های واقعی در GitHub یا وبلاگ‌های Vercel/Netlify |

---