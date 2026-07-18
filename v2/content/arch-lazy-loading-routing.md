<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "Lazy Loading در سطح Routing (prefetching و تأثیر بر تجربه‌ی کاربر)" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**Lazy Loading در Routing** — بارگذاری صفحات فقط زمانی که کاربر به آن مسیر navigates کند. در Next.js App Router, route-based code splitting خودکار است — هر `page.tsx` یک chunk جدا.

‏**Prefetching** — بارگذاری صفحه در background قبل از کلیک کاربر. Next.js به صورت پیش‌فرض صفحات visible در viewport را prefetch می‌کند (در production).

</div>

```javascript
// Next.js automatic route-level code splitting
app/
  page.tsx          → / (home chunk)
  about/page.tsx    → /about → chunk only when navigated
  dashboard/page.tsx → /dashboard → separate chunk

// Link prefetching (default)
<Link href="/about">About</Link>
// Next.js automatically prefetches /about chunk
// when Link enters viewport (idle time)

// Disable prefetch (rare pages)
<Link href="/settings" prefetch={false}>Settings</Link>
// Only when user clicks

// Manual prefetch
const router = useRouter();
const prefetchPage = useCallback(() => {
  router.prefetch('/dashboard');
}, [router]);

// Smart prefetch on hover
const handleHover = useCallback(() => {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = '/dashboard';
  document.head.appendChild(link);
}, []);
```

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏| استراتژی | زمان بارگذاری | Bandwidth | موارد استفاده |
‏|----------|---------------|-----------|---------------|
‏| No prefetch | بعد از کلیک | ۰ | صفحات heavy (admin, reports) |
‏| Default prefetch (viewport) | بعد از viewport entry | متوسط | صفحات عمومی |
‏| Prefetch on hover | بعد از hover | کم | صفحات CTA (checkout, signup) |
‏| Eager prefetch | بلافاصله بعد از load | زیاد | صفحات بعدی در funnel |

</div>

```javascript
// x استراتژی‌های prefetch

// x ۱. Default: viewport-based (ساده)
<Link href="/products">Products</Link>
// Next.js prefetches when link in viewport

// x ۲. Priority prefetch (critical path)
function Navigation() {
  return (
    <nav>
      <Link href="/" prefetch={true}>Home</Link>      {/* high priority */}
      <Link href="/products">Products</Link>            {/* viewport */}
      <Link href="/checkout" prefetch={false}>Checkout</Link> {/* on click only */}
    </nav>
  );
}

// x ۳. Predictive prefetch (after certain actions)
function ProductPage({ productId }) {
  const router = useRouter();

  useEffect(() => {
    // After viewing product for 3s, prefetch checkout
    const timer = setTimeout(() => {
      router.prefetch('/checkout');
    }, 3000);
    return () => clearTimeout(timer);
  }, [router]);

  return <ProductDetail />;
}

// x ۴. Route-based prefetching
function useSmartPrefetch(routes: string[]) {
  const router = useRouter();
  const prefetched = useRef(new Set());

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            const route = entry.target.getAttribute('data-route');
            if (route && !prefetched.current.has(route)) {
              router.prefetch(route);
              prefetched.current.add(route);
            }
          }
        });
      },
      { rootMargin: '200px' } // prefetch 200px before viewport
    );
    // ... observe Links
  }, [router, routes]);
}
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏**سناریو:** یک سایت با ۵۰+ صفحه و ۵ زبان. prefetch همه زبان‌ها → bandwidth هدر.

‏**چالش:** prefetch پیش‌فرض Next.js همه pages visible Links را prefetch می‌کند. اگر صفحه ۲۰۰ Link داشته باشد, prefetch storm → bandwidth مصرفی بالا در موبایل.

‏**راه‌حل:** Priority-based prefetch + bandwidth-aware + only current locale.

</div>

```javascript
// Bandwidth-aware prefetch
function BandwidthAwareLink({ href, children }) {
  const router = useRouter();
  const connection = useMemo(() => {
    if (typeof navigator === 'undefined') return null;
    return navigator.connection;
  }, []);

  const shouldPrefetch = useCallback(() => {
    if (!connection) return true;

    // Don't prefetch on slow connections
    if (connection.effectiveType === 'slow-2g' || connection.effectiveType === '2g') {
      return false;
    }
    // Don't prefetch on data saver
    if ((connection as any).saveData) return false;
    return true;
  }, [connection]);

  const handleMouseEnter = useCallback(() => {
    if (shouldPrefetch()) {
      router.prefetch(href);
    }
  }, [router, href, shouldPrefetch]);

  return (
    <Link
      href={href}
      prefetch={false} // manual control
      onMouseEnter={handleMouseEnter}
    >
      {children}
    </Link>
  );
}

// x Locale-aware prefetch — فقط زبان فعلی
function LocaleAwareLink({ href, locale, children }) {
  const currentLocale = useLocale();
  if (locale !== currentLocale) {
    return <Link href={href} prefetch={false}>{children}</Link>;
  }
  return <Link href={href}>{children}</Link>;
}

// Performance monitoring
// Network tab: filter "prefetch" requests
// Measure: prefetch hit rate = pages loaded from prefetch cache / total navigations
// x هدف: prefetch hit rate > 80%
```

<div dir="rtl">

‏**نکته Performance:** prefetch مناسب می‌تواند page transition time را از ۲-۳s به < 100ms کاهش دهد (instant navigation). اما prefetch بی‌رویه bandwidth و battery مصرف می‌کند.

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏بدون Route-level Lazy Loading:

</div>

| روش | محدودیت |
|-----|---------|
| Single bundle | Large initial load, slow first paint |
| Manual dynamic imports | Non-declarative, no prefetch |
| Client-side routing (CSR) | No SSR, slow first page |

<div dir="rtl">

‏**مقایسه Next.js vs Express.js:** Next.js App Router route-based splitting خودکار. Express.js manual `React.lazy` + `React Router` نیاز دارد. prefetch در Express.js + React Router با `@loadable/component` یا `React.lazy` manual است.

</div>

<div dir="rtl">

‏> **نهایی:** Lazy Loading در Routing ساده‌ترین win performance است — zero-config در App Router. prefetch smart انجام بده: default برای pages مهم, disabled برای heavy pages, bandwidth-aware برای موبایل. مهم‌ترین نکته Senior: prefetch storm را با `IntersectionObserver` + `rootMargin` کنترل کن — ۲۰۰px قبل از viewport کافی است.

</div>
