<div dir="rtl">
# راهنمای جامع Server Actions، Cache Components و فرم‌ها در Next.js 16

> این مقاله مرجع کدهای همین پروژه است. تمام مباحث عملی‌شده در سیستم **ساخت / ویرایش / حذف پست** با **Server Actions** روی مدل **Cache Components** آورده شده‌اند. هدف: خواندن این مقاله و یادگیری کامل جریان کار.

---

## فهرست مطالب

1. [پیش‌نیاز: فعال کردن Cache Components](#۱-پیش‌نیاز-فعال-کردن-cache-components)
2. [مسیر کامل داده: از فرم تا ذخیره](#۲-مسیر-کامل-داده-از-فرم-تا-ذخیره)
3. [لایهٔ داده: اعتبارسنجی دوگانه](#۳-لایهٔ-داده-اعتبارسنجی-دوگانه)
4. [لایهٔ خوانش کش‌شده: use cache + cacheLife + cacheTag](#۴-لایهٔ-خوانش-کش‌شده-use-cache--cachelife--cachetag)
5. [API Routes: اضافه کردن POST، PUT، DELETE](#۵-api-routes-اضافه-کردن-postputdelete)
6. [Server Actions: اکشن‌های سمت سرور](#۶-server-actionsاکشن‌های-سمت-سرور)
7. [کامپوننت‌های کلاینت: فرم و دکمه‌ها](#۷-کامپوننت‌های-کلاینت-فرم-و-دکمه‌ها)
8. [سیستم بارگذاری: Suspense، loading.tsx، Skeleton](#۸-سیستم-بارگذاری-suspenseloading.tsxskeleton)
9. [read-your-own-writes: updateTag در برابر revalidateTag](#۹-read-your-own-writes-updatetag-در-برابر-revalidatetag)
10. [نقشهٔ کامل فایل‌ها](#۱۰-نقشهٔ-کامل-فایل‌ها)
11. [خروجی build و نقشهٔ مسیرها](#۱۱-خروجی-build-و-نقشهٔ-مسیرها)
12. [آزمایش و تأیید عملکرد](#۱۲-آزمایش-و-تأیید-عملکرد)
13. [مشکلات رایج و نکات مهم](#۱۳-مشکلات-رایج-و-نکات-مهم)

---

## ۱. پیش‌نیاز: فعال کردن Cache Components

اولین قدم، روشن کردن مدل Cache Components در `next.config.ts` است. بدون آن، دایرکتیو `use cache`، توابع `cacheLife`/`cacheTag` و `updateTag` هیچ‌کدام کار نمی‌کنند.

<div dir="ltr">

```ts
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // Stable since Next.js 16.0 — not experimental.
  // Enables: use cache directive, PPR, cacheTag, cacheLife, updateTag
  cacheComponents: true,
};

export default nextConfig;
```

</div>

### چه اتفاقی می‌افتد؟

با فعال شدن این تنظیم:

| قابلیت | توضیح |
|--------|-------|
| `export const dynamic` / `revalidate` | حذف شده‌اند — دیگر نمی‌توانید `dynamic = "force-dynamic"` بنویسید |
| `use cache` دایرکتیو | روی تابع سرور اعمال می‌شود و خروجی را کش می‌کند |
| `cacheLife("seconds")` | عمر کش را تعیین می‌کند |
| `cacheTag("posts")` | برچسبی برای invalidate گروهی |
| `updateTag("posts")` | فقط در Server Action — تازه‌سازی همان‌roundtrip |
| PPR (Partial Prerender) | مدل رندر پیش‌فرض: پوستهٔ استاتیک + محتوای پویای استریم‌شده |

---

## ۲. مسیر کامل داده: از فرم تا ذخیره

### قانون طلایی پروژه

> **هیچ کامپوننت یا اکشنی مستقیماً به لایهٔ داده دست نمی‌زند.**

همه‌چیز از مسیر API رد می‌شود. این یعنی هر تغییری دوبار اعتبارسنجی می‌شود: یک بار در اکشن (خطای سریع) و یک بار در API (پشتیبان امنیتی).

### جریان کامل

```
┌──────────────────────────────────────────────────────────────┐
│                  کامپوننت سرور (صفحه)                         │
│         app/posts/new/page.tsx یا app/posts/[id]/edit/page.tsx │
│                                                              │
│   PostForm را با یک اکشن سرور render می‌کند                   │
│         ↓                                                    │
├──────────────────────────────────────────────────────────────┤
│                  فرم سمت کلاینت                                │
│              components/PostForm.tsx                          │
│              "use client"                                     │
│                                                              │
│   useActionState(action, { ok: true })                       │
│   کاربر دکمه را می‌زند → FormData ارسال می‌شود                 │
│         ↓                                                    │
├──────────────────────────────────────────────────────────────┤
│                Server Action                                  │
│              app/posts/actions.ts                             │
│              "use server"                                     │
│                                                              │
│   1. parsePostInput() — اعتبارسنجی سریع                       │
│   2. fetch → API route                                       │
│   3. updateTag('posts') — کش را باطل می‌کند                   │
│   4. redirect() — کاربر را به صفحه هدایت می‌کند                │
│         ↓                                                    │
├──────────────────────────────────────────────────────────────┤
│                API Route                                      │
│              app/api/posts/route.ts                           │
│                                                              │
│   1. parsePostInput() — پشتیبان دوم                           │
│   2. createPost() / updatePost() / deletePost()              │
│         ↓                                                    │
├──────────────────────────────────────────────────────────────┤
│                لایهٔ داده                                      │
│              lib/posts.ts                                     │
│              data/posts.json                                  │
└──────────────────────────────────────────────────────────────┘
```

### چرا دوبار اعتبارسنجی؟

در معماری وب واقعی، API عمومی است و هر کسی می‌تواند مستقیماً به آن درخواست بفرستد. بنابراین:

- **اکشن** (مرز اول): اعتبارسنجی سریع انجام می‌دهد تا کاربر بلافاصله خطا ببیند و مجبور نباشد منتظر roundtrip سرور بماند.
- **API** (مرز دوم): پشتیبان امنیتی — اگر کسی مستقیماً API را صدا زد (بدون اکشن)، باز هم اعتبارسنجی انجام شود.

---

## ۳. لایهٔ داده: اعتبارسنجی دوگانه

### `parsePostInput`: تابع اعتبارسنجی مشترک

این تابع در `lib/posts.ts` تعریف شده و **هم توسط اکشن‌ها و هم توسط API route ها** صدا زده می‌شود:

<div dir="ltr">

```ts
// lib/posts.ts

export interface PostInput {
  title: string;
  content: string;
  author: string;
}

export type PostErrors = Partial<
  Record<"title" | "content" | "author" | "form", string>
>;

export type ParsePostResult =
  | { ok: true; data: PostInput }
  | { ok: false; errors: PostErrors };

export function parsePostInput(raw: Partial<PostInput>): ParsePostResult {
  const title = typeof raw.title === "string" ? raw.title.trim() : "";
  const content = typeof raw.content === "string" ? raw.content.trim() : "";
  const author = typeof raw.author === "string" ? raw.author.trim() : "";

  const errors: PostErrors = {};
  if (!title) errors.title = "عنوان الزامی است.";
  if (!content) errors.content = "محتوا نمی‌تواند خالی باشد.";
  if (!author) errors.author = "نام نویسنده الزامی است.";

  if (Object.keys(errors).length > 0) return { ok: false, errors };
  return { ok: true, data: { title, content, author } };
}
```

</div>

### توابع CRUD

<div dir="ltr">

```ts
// lib/posts.ts — ادامه

export async function createPost(input: PostInput): Promise<Post> {
  const posts = await getPosts();
  const id = posts.reduce((max, p) => Math.max(max, p.id), 0) + 1;
  const post: Post = {
    id,
    title: input.title,
    slug: slugify(input.title, id), // اگر عنوان فارسی باشد: post-{id}
    content: input.content,
    author: input.author,
    publishedAt: new Date().toISOString().slice(0, 10),
    views: 0,
  };
  posts.unshift(post);
  await writePosts(posts);
  return post;
}

export async function updatePost(
  id: string,
  input: PostInput
): Promise<Post | undefined> {
  const posts = await getPosts();
  const index = posts.findIndex((p) => String(p.id) === id);
  if (index === -1) return undefined; // پست وجود ندارد

  const existing = posts[index];
  posts[index] = {
    ...existing,           // id، slug، views، publishedAt حفظ می‌شوند
    title: input.title,
    slug: slugify(input.title, existing.id),
    content: input.content,
    author: input.author,
  };
  await writePosts(posts);
  return posts[index];
}

export async function deletePost(id: string): Promise<boolean> {
  const posts = await getPosts();
  const next = posts.filter((p) => String(p.id) !== id);
  if (next.length === posts.length) return false; // چیزی حذف نشد
  await writePosts(next);
  return true;
}
```

</div>

### نکتهٔ slug برای عناوین فارسی

تابع `slugify` سعی می‌کند از عنوان، slug لاتین بسازد. اگر عنوان کاملاً فارسی باشد (هیچ حرف لاتینی نداشته باشد)، به `post-{id}` سقوط می‌کند:

<div dir="ltr">

```ts
function slugify(title: string, id: number): string {
  const latin = title
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/^-+|-+$/g, "");
  return latin || `post-${id}`;
}
```

</div>

---

## ۴. لایهٔ خوانش کش‌شده: `use cache` + `cacheLife` + `cacheTag`

### فایل `lib/post-data.ts`

این فایل **جای خوانش‌های مستقیم `fetch` در صفحات را می‌گیرد**. به جای اینکه هر صفحه مستقیماً API را fetch کند، از این توابع کش‌شده استفاده می‌شود:

<div dir="ltr">

```ts
// lib/post-data.ts
import { cacheLife, cacheTag } from "next/cache";
import type { Post } from "@/lib/posts";

const API_URL = process.env.APP_URL || "http://localhost:3000";

export async function getPostsCached(): Promise<Post[]> {
  "use cache";               // ← دایرکتیو: خروجی کش می‌شود
  cacheLife("seconds");       // ← عمر کش
  cacheTag("posts");          // ← برچسب برای invalidate گروهی
  try {
    const res = await fetch(`${API_URL}/api/posts`, { cache: "no-store" });
    if (!res.ok) return [];
    return (await res.json()) as Post[];
  } catch {
    return []; // fallback نرم: بدون API، لیست خالی
  }
}

export async function getPostCached(id: string): Promise<Post | undefined> {
  "use cache";
  cacheLife("seconds");
  cacheTag("posts");
  try {
    const res = await fetch(`${API_URL}/api/posts/${id}`, {
      cache: "no-store",
    });
    if (!res.ok) return undefined;
    return (await res.json()) as Post;
  } catch {
    return undefined; // fallback نرم: بدون API → notFound()
  }
}
```

</div>

### هر خط چه می‌کند؟

| خط | نقش |
|-----|------|
| `"use cache"` | خروجی تابع در حافظهٔ کش Next.js ذخیره می‌شود. اگر همان ورودی دوبار صدا زده شود، نتیجهٔ کش‌شده برمی‌گردد. |
| `cacheLife("seconds")` | تعیین می‌کند کش چقدر زنده بماند. اگر بیرون از تابع `use cache` صدا زده شود، **throw** می‌کند. |
| `cacheTag("posts")` | برچسبی که `updateTag("posts")` در اکشن‌ها برای باطل کردن آن استفاده می‌کند. تمام خوانش‌هایی که `cacheTag("posts")` دارند، با فراخوانی `updateTag("posts")` باطل می‌شوند. |
| `cache: "no-store"` | به `fetch` می‌گوید پاسخ را در کش مرورگر/SDK ذخیره نکند — چون ما خودمان کش می‌کنیم. |
| `try/catch` + fallback | اگر API در دسترس نباشد (مثلاً در زمان build بدون سرور)، به جای crash شدن، `[]` یا `undefined` برمی‌گردد. |

### رفتار قبل و بعد

| قبل (بدون cacheComponents) | بعد (با cacheComponents) |
|---------------------------|----------------------|
| `fetch(API, { cache: "no-store" })` — هر درخواست، رندر کامل سمت سرور | `getPostsCached()` — کش با عمر "seconds"، invalidate با `updateTag` |
| تازگی داده در هر درخواست | تازگی با `updateTag` در همان roundtrip |

---

## ۵. API Routes: اضافه کردن POST، PUT، DELETE

### `POST /api/posts` — ساخت پست

<div dir="ltr">

```ts
// app/api/posts/route.ts
import { NextResponse } from "next/server";
import { createPost, getPosts, parsePostInput } from "@/lib/posts";

export async function GET() {
  const posts = await getPosts();
  return NextResponse.json(posts, {
    headers: { "x-server-time": new Date().toISOString() },
  });
}

export async function POST(req: Request) {
  // مرز دوم اعتبارسنجی (پشتیبان)
  const parsed = parsePostInput(await req.json().catch(() => ({})));
  if (!parsed.ok) {
    return NextResponse.json({ errors: parsed.errors }, { status: 400 });
  }

  const post = await createPost(parsed.data);
  return NextResponse.json(post, { status: 201 });
}
```

</div>

### `PUT` و `DELETE /api/posts/[id]`

<div dir="ltr">

```ts
// app/api/posts/[id]/route.ts
import { NextResponse } from "next/server";
import {
  deletePost,
  getPost,
  parsePostInput,
  updatePost,
} from "@/lib/posts";

export async function GET(
  _req: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  // تأخیر شبیه‌سازی‌شده — فقط در GET فعال است
  await new Promise((resolve) => setTimeout(resolve, 1500));
  const post = await getPost(id);
  if (!post) {
    return NextResponse.json({ error: "Post not found" }, { status: 404 });
  }
  return NextResponse.json(post, {
    headers: { "x-server-time": new Date().toISOString() },
  });
}

export async function PUT(
  req: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const parsed = parsePostInput(await req.json().catch(() => ({})));
  if (!parsed.ok) {
    return NextResponse.json({ errors: parsed.errors }, { status: 400 });
  }
  const post = await updatePost(id, parsed.data);
  if (!post) {
    return NextResponse.json({ error: "Post not found" }, { status: 404 });
  }
  return NextResponse.json(post, { status: 200 });
}

export async function DELETE(
  _req: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const deleted = await deletePost(id);
  if (!deleted) {
    return NextResponse.json({ error: "Post not found" }, { status: 404 });
  }
  return new NextResponse(null, { status: 204 });
}
```

</div>

### نکتهٔ `params: Promise<{ id: string }>`

در Next.js 16، پارامترهای مسیر پویا (`[id]`) به صورت **Promise** وارد می‌شوند و باید `await` شوند. این الگو قبلاً هم در `GET` موجود استفاده شده بود:

<div dir="ltr">

```ts
// الگوی قدیمی (Next.js 14-15):
export async function GET(
  _req: Request,
  { params }: { params: { id: string } }   // sync
) {
  const id = params.id;                      // بدون await
}

// الگوی جدید (Next.js 16):
export async function GET(
  _req: Request,
  { params }: { params: Promise<{ id: string }> }   // async
) {
  const { id } = await params;                       // با await
}
```

</div>

### قرارداد کدهای وضعیت

| درخواست | موفق | خطا |
|---------|------|-----|
| `POST /api/posts` | `201` — پست ساخته‌شده | `400` — خطا در اعتبارسنجی |
| `PUT /api/posts/[id]` | `200` — پست به‌روزرسانی‌شده | `400` / `404` |
| `DELETE /api/posts/[id]` | `204` — بدون بدنه | `404` |

---

## ۶. Server Actions: اکشن‌های سمت سرور

### ساختار کلی

<div dir="ltr">

```ts
// app/posts/actions.ts
"use server";

import { redirect } from "next/navigation";
import { updateTag } from "next/cache";
import { parsePostInput } from "@/lib/posts";
import type { PostErrors } from "@/lib/posts";

export type FormState = {
  ok: boolean;
  errors?: PostErrors;
};

const API_URL = process.env.APP_URL || "http://localhost:3000";
```

</div>

### ساخت پست

<div dir="ltr">

```ts
export async function createPostAction(
  _prev: FormState,
  fd: FormData
): Promise<FormState> {
  // 1. اعتبارسنجی سریع (مرز اول)
  const parsed = parsePostInput({
    title: fd.get("title")?.toString() ?? "",
    content: fd.get("content")?.toString() ?? "",
    author: fd.get("author")?.toString() ?? "",
  });
  if (!parsed.ok) return { ok: false, errors: parsed.errors };

  // 2. ارسال به API
  const res = await fetch(`${API_URL}/api/posts`, {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify(parsed.data),
  });

  // 3. اگر API خطا برگرداند
  if (!res.ok) {
    const body = (await res.json().catch(() => ({}))) as {
      errors?: PostErrors;
    };
    return {
      ok: false,
      errors: body.errors ?? { form: "خطا در ثبت پست؛ دوباره تلاش کنید." },
    };
  }

  // 4. باطل کردن کش — حتماً قبل از redirect
  updateTag("posts");

  // 5. هدایت کاربر
  redirect("/posts");
}
```

</div>

### ویرایش پست

<div dir="ltr">

```ts
export async function updatePostAction(
  _prev: FormState,
  fd: FormData
): Promise<FormState> {
  const id = fd.get("id")?.toString();
  if (!id) return { ok: false, errors: { form: "شناسه پست نامعتبر است." } };

  const parsed = parsePostInput({
    title: fd.get("title")?.toString() ?? "",
    content: fd.get("content")?.toString() ?? "",
    author: fd.get("author")?.toString() ?? "",
  });
  if (!parsed.ok) return { ok: false, errors: parsed.errors };

  const res = await fetch(`${API_URL}/api/posts/${id}`, {
    method: "PUT",
    headers: { "content-type": "application/json" },
    body: JSON.stringify(parsed.data),
  });

  if (!res.ok) {
    const body = (await res.json().catch(() => ({}))) as {
      errors?: PostErrors;
    };
    return {
      ok: false,
      errors: body.errors ?? { form: "خطا در ذخیره پست؛ دوباره تلاش کنید." },
    };
  }

  updateTag("posts");
  redirect(`/posts/${id}`);
}
```

</div>

### حذف پست

<div dir="ltr">

```ts
// نکته: حذف مستقیماً به عنوان action فرم استفاده می‌شود
// (نه از طریق useActionState)، پس return type باید Promise<void> باشد
export async function deletePostAction(fd: FormData): Promise<void> {
  const id = fd.get("id")?.toString();
  if (!id) return;

  const res = await fetch(`${API_URL}/api/posts/${id}`, {
    method: "DELETE",
  });
  if (!res.ok) return;

  updateTag("posts");
  redirect("/posts");
}
```

</div>

### سه قاعدهٔ طلایی در اکشن‌ها

**قاعدهٔ ۱: `updateTag` حتماً قبل از `redirect`**

<div dir="ltr">

```ts
// ✅ درست
updateTag("posts");   // اول: کش را باطل کن
redirect("/posts");   // بعد: redirect (پرتاب استثنا)

// ❌ اشتباه
redirect("/posts");   // اول: throw می‌کند ← کد بعدی هرگز اجرا نمی‌شود
updateTag("posts");   // هرگز اجرا نمی‌شود ← کاربر لیست کهنه می‌بیند
```

</div>

چرا؟ چون `redirect` یک **استثنا** پرتاب می‌کند. هیچ خطی بعد از `redirect` اجرا نمی‌شود.

**قاعدهٔ ۲: `deletePostAction` باید `Promise<void>` برگرداند**

چون مستقیماً به `<form action={deletePostAction}>` متصل است (نه از طریق `useActionState`)، نوع برگشتی باید `void | Promise<void>` باشد. اگر `Promise<FormState>` برگردانید، TypeScript خطا می‌دهد.

**قاعدهٔ ۳: خطای API را مستند کنید، throw نکنید**

<div dir="ltr">

```ts
// ✅ درست — خطا را به FormState برگردانید
if (!res.ok) {
  const body = (await res.json().catch(() => ({})));
  return { ok: false, errors: body.errors ?? { form: "خطا..." } };
}

// ❌ اشتباه — throw کردن باعث crash صفحه می‌شود
if (!res.ok) throw new Error("API failed");
```

</div>

---

## ۷. کامپوننت‌های کلاینت: فرم و دکمه‌ها

### `PostForm.tsx` — فرم مشترک ساخت/ویرایش

<div dir="ltr">

```tsx
"use client";

import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import type { FormState } from "@/app/posts/actions";

export interface PostFormValues {
  id?: string;
  title?: string;
  content?: string;
  author?: string;
}

function SubmitButton({ label }: { label: string }) {
  const { pending } = useFormStatus();
  return (
    <button
      type="submit"
      disabled={pending}
      className="..."
    >
      {pending ? "در حال ذخیره…" : label}
    </button>
  );
}

export default function PostForm({
  action,
  initialValues,
  submitLabel,
}: {
  action: (prev: FormState, fd: FormData) => Promise<FormState>;
  initialValues?: PostFormValues;
  submitLabel: string;
}) {
  const [state, formAction] = useActionState(action, { ok: true });

  return (
    <form action={formAction} dir="rtl">
      {initialValues?.id && (
        <input type="hidden" name="id" value={initialValues.id} />
      )}

      {state.errors?.form && (
        <p className="text-red-400">{state.errors.form}</p>
      )}

      <label htmlFor="title">عنوان</label>
      <input
        id="title"
        name="title"
        defaultValue={initialValues?.title ?? ""}
      />
      {state.errors?.title && (
        <p className="text-red-400">{state.errors.title}</p>
      )}

      {/* ... author و content مشابه ... */}

      <SubmitButton label={submitLabel} />
    </form>
  );
}
```

</div>

### `useActionState`: چگونه کار می‌کند

<div dir="ltr">

```tsx
const [state, formAction] = useActionState(action, { ok: true });
//            ↑                ↑
//     تابع اکشن سرور    مقدار اولیه state
//     (prev, fd) => FormState
```

</div>

| پارامتر | نقش |
|---------|------|
| `action` | تابع سرور: `(prevState, formData) => Promise<FormState>` |
| `{ ok: true }` | مقدار اولیه — state قبل از اولین submit |
| `state` | نتیجهٔ آخرین اجرای اکشن (شامل `errors`) |
| `formAction` | تابعی که به `action` prop فرم متصل می‌شود |

### چرا `defaultValue` به جای `value`؟

```tsx
// ✅ incontrol — متن کاربر بعد از خطا حفظ می‌شود
<input name="title" defaultValue={initialValues?.title ?? ""} />

// ❌ control — مقدار از server می‌آید، متن تایپ‌شده بعد از submit ناپدید می‌شود
<input name="title" value={state.title} onChange={...} />
```

با `defaultValue`، اگر submit با خطا برگردد، متن تایپ‌شدهٔ کاربر **حفظ می‌شود** بدون نیاز به roundtrip کردن مقادیر فیلدها.

### `useFormStatus`: وضعیت pending

<div dir="ltr">

```tsx
function SubmitButton({ label }: { label: string }) {
  const { pending } = useFormStatus();
  return (
    <button disabled={pending}>
      {pending ? "در حال ذخیره…" : label}
    </button>
  );
}
```

</div>

> ⚠️ نکته: `useFormStatus` باید در یک **فرزند جداگانه** استفاده شود (نه در همان کامپوننتی که `useActionState` دارد). چون `useFormStatus` والد خود را چک می‌کند — اگر در همان فرم باشد، فرم والد خودش را پیدا می‌کند.

### `DeletePostButton.tsx`

<div dir="ltr">

```tsx
"use client";

import { useFormStatus } from "react-dom";
import { deletePostAction } from "@/app/posts/actions";

function DeleteButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "در حال حذف…" : "حذف پست"}
    </button>
  );
}

export default function DeletePostButton({ postId }: { postId: string }) {
  return (
    <form
      action={deletePostAction}
      onSubmit={(e) => {
        if (!confirm("این پست برای همیشه حذف شود؟")) e.preventDefault();
      }}
    >
      <input type="hidden" name="id" value={postId} />
      <DeleteButton />
    </form>
  );
}
```

</div>

نکات:
- `confirm()` قبل از ارسال — کاربر باید تأیید کند
- `deletePostAction` مستقیماً به `action` متصل است (بدون `useActionState`)
- `return type` اکشن باید `Promise<void>` باشد (نه `Promise<FormState>`)

---

## ۸. سیستم بارگذاری: Suspense، loading.tsx، Skeleton

### ماتریس بارگذاری

| لایه | ابزار | مسئولیت |
|------|-------|---------|
| دکمهٔ داخل فرم | `useFormStatus` | disable کردن دکمه + «در حال ذخیره…» |
| خطاهای اعتبارسنجی | `useActionState` | نمایش خطا زیر فیلدها بدون از دست رفتن متن |
| دادهٔ صفحه (مثلاً edit) | `Suspense` + `loading.tsx` | اسکلتون در زمان منتظر ماندن داده |
| ناوبری سطح مسیر | `loading.tsx` | فallback بارگذاری سطح مسیر |

### چرا `Suspense` الزامی است؟

تحت `cacheComponents`، وقتی داده از کش می‌آید و کش **miss** شده باشد، داده «uncached» است. Next.js اجازه نمی‌دهد دادهٔ uncached **خارج از `<Suspense>`** باشد — چون صفحه را بلاک می‌کند و تجربهٔ کاربری کند می‌شود.

اگر `<Suspense>` نگذارید، build با این خطا متوقف می‌شود:

```
Error: Route "/posts": Uncached data was accessed outside of <Suspense>.
This delays the entire page from rendering, resulting in a slow user experience.
```

### الگوی صحیح: فرزند جداگانه تحت Suspense

**صفحهٔ ویرایش (`app/posts/[id]/edit/page.tsx`):**

<div dir="ltr">

```tsx
import { Suspense } from "react";
import { notFound } from "next/navigation";
import PostForm from "@/components/PostForm";
import FormSkeleton from "@/components/FormSkeleton";
import { updatePostAction } from "@/app/posts/actions";
import { getPostCached } from "@/lib/post-data";

// فرزند جداگانه — داده‌گیری اینجا انجام می‌شود
async function EditForm({ id }: { id: string }) {
  const post = await getPostCached(id);
  if (!post) notFound();

  return (
    <PostForm
      action={updatePostAction}
      initialValues={{
        id: String(post.id),
        title: post.title,
        content: post.content,
        author: post.author,
      }}
      submitLabel="ذخیره تغییرات"
    />
  );
}

export default async function EditPostPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  return (
    <div className="space-y-6">
      <h1>ویرایش پست</h1>
      <Suspense fallback={<FormSkeleton />}>
        <EditForm id={id} />
      </Suspense>
    </div>
  );
}
```

</div>

> ⚠️ نکته: `getPostCached(id)` با تأخیر شبیه‌سازی‌شدهٔ ۱.۵ ثانیه‌ای (در `GET /api/posts/[id]`) اجرا می‌شود. بدون `<Suspense>`، کل صفحه ۱.۵ ثانیه بلاک می‌شود. با `<Suspense>`، ابتدا پوستهٔ اسکلتون نمایش داده و سپس محتوا **استریم** می‌شود.

**صفحهٔ لیست (`app/posts/page.tsx`):**

<div dir="ltr">

```tsx
import { Suspense } from "react";
import Link from "next/link";
import PostCard from "@/components/PostCard";
import { getPostsCached } from "@/lib/post-data";

async function PostList() {
  const posts = await getPostsCached();
  return (
    <>
      <Link href="/posts/new">پست جدید</Link>
      <div className="grid gap-4 sm:grid-cols-2">
        {posts.map((post) => (
          <PostCard key={post.id} post={post} basePath="/posts" />
        ))}
      </div>
    </>
  );
}

export default async function PostsPage() {
  return (
    <div className="space-y-6">
      <h1>فهرست محصولات</h1>
      <Suspense fallback={<div>در حال بارگذاری...</div>}>
        <PostList />
      </Suspense>
    </div>
  );
}
```

</div>

### `loading.tsx`: فallback سطح مسیر

هر پوشه می‌تواند فایل `loading.tsx` داشته باشد. این فایل **هنگام ناوبری به آن مسیر** نمایش داده می‌شود (قبل از load شدن صفحه):

<div dir="ltr">

```tsx
// app/posts/new/loading.tsx
import FormSkeleton from "@/components/FormSkeleton";

export default function Loading() {
  return <FormSkeleton />;
}

// app/posts/[id]/edit/loading.tsx — دقیقاً یکسان
import FormSkeleton from "@/components/FormSkeleton";

export default function Loading() {
  return <FormSkeleton />;
}
```

</div>

### `FormSkeleton`: اسکلتون مشترک

<div dir="ltr">

```tsx
// components/FormSkeleton.tsx
export default function FormSkeleton() {
  return (
    <div className="mx-auto max-w-2xl space-y-4" aria-hidden>
      <div className="h-6 w-1/4 animate-pulse rounded bg-gray-800" />
      <div className="space-y-1">
        <div className="h-4 w-16 rounded bg-gray-800" />
        <div className="h-9 w-full animate-pulse rounded bg-gray-800" />
      </div>
      <div className="space-y-1">
        <div className="h-4 w-16 rounded bg-gray-800" />
        <div className="h-9 w-full animate-pulse rounded bg-gray-800" />
      </div>
      <div className="space-y-1">
        <div className="h-4 w-16 rounded bg-gray-800" />
        <div className="h-36 w-full animate-pulse rounded bg-gray-800" />
      </div>
      <div className="h-9 w-28 rounded bg-gray-800" />
    </div>
  );
}
```

</div>

### جریان بارگذاری کامل

```
کلیک کاربر روی لینک "ویرایش"
         │
         ▼
loading.tsx اجرا می‌شود (اسکلتون سطح مسیر)
         │
         ▼
صفحه edit رندر می‌شود
         │
         ▼
Suspense fallback: FormSkeleton (اسکلتون داخل صفحه)
         │
         ▼
getPostCached(id) → cache miss → fetch API → ۱.۵ ثانیه تأخیر
         │
         ▼
محتوا استریم می‌شود → PostForm با داده‌های پر شده
```

---

## ۹. read-your-own-writes: `updateTag` در برابر `revalidateTag`

### جدول مقایسه

| ابزار | کجا کار می‌کند | رفتار | کِی تازه می‌شود |
|-------|----------------|-------|----------------|
| `updateTag("posts")` | فقط Server Action | **read-your-own-writes** | **همان roundtrip** — بعد از redirect، صفحهٔ مقصد دادهٔ تازه را می‌بیند |
| `revalidateTag("posts")` | Route handler / webhook / هر جا | **stale-while-revalidate** | درخواست فعلی ممکن است دادهٔ قدیمی برگرداند؛ بازآوری در پس‌زمینه |
| `revalidatePath("/posts")` | Server Action / Route Handler | حذف کل مسیر از کش | مشابه revalidateTag ولی به مسیر نه برچسب |

### مثال عملی: تفاوت `updateTag` و `revalidateTag`

فرض کنید کاربر پست جدیدی ساخته و به صفحهٔ لیست برمی‌گردد:

**با `updateTag`:**
```
action: createPost → DB → updateTag("posts") → redirect("/posts")
                                                          │
                                      صفحهٔ /posts لود می‌شود
                                      getPostsCached() → کش miss (باطل شده)
                                      → fetch API → پست جدید در لیست ✅
```

**با `revalidateTag`:**
```
action: createPost → DB → revalidateTag("posts") → redirect("/posts")
                                                          │
                                      صفحهٔ /posts لود می‌شود
                                      getPostsCached() → کش ممکن است هنوز قدیمی باشد
                                      → پست جدید ممکن است نباشد ❌ (تا roundtrip بعدی)
```

### محدودیت کلاینت

روتر کلاینت Next.js برای محتوای `use cache` **حداقل ۳۰ ثانیه stale** را اجرا می‌کند. یعنی اگر کاربر صفحه را رفرش کند، ممکن است ۳۰ ثانیه دادهٔ کهنه ببیند. `updateTag` این محدودیت را دور می‌زند چون در **همان درخواست** رندر را تازه می‌کند.

### چرا `use cache` و `GET` ناسازگارند؟

`use cache` را نمی‌توان مستقیماً روی خود export `GET` در route handler اعمال کرد. در عuzz، handler تابعِ `use cache` را صدا می‌زند (دقیقاً همان کاری که `getPostsCached` می‌کند).

---

## ۱۰. نقشهٔ کامل فایل‌ها

| فایل | نوع | نقش |
|------|------|-----|
| `next.config.ts` | تغییر | `cacheComponents: true` |
| `lib/posts.ts` | تغییر | داده + CRUD + `parsePostInput` |
| `lib/post-data.ts` | **جدید** | خوانش کش‌شده (`use cache` + `cacheLife` + `cacheTag`) |
| `app/api/posts/route.ts` | تغییر | `GET` + `POST` |
| `app/api/posts/[id]/route.ts` | تغییر | `GET` + `PUT` + `DELETE` |
| `app/posts/actions.ts` | **جدید** | سه اکشن + `FormState` type |
| `components/PostForm.tsx` | **جدید** | فرم مشترک ساخت/ویرایش (کلاینت) |
| `components/DeletePostButton.tsx` | **جدید** | دکمهٔ حذف با confirm |
| `components/FormSkeleton.tsx` | **جدید** | اسکلتون مشترک |
| `app/posts/new/page.tsx` | **جدید** | فرم ثبت (پوستهٔ استاتیک) |
| `app/posts/[id]/edit/page.tsx` | **جدید** | فرم ویرایش (داینامیک + Suspense) |
| `app/posts/new/loading.tsx` | **جدید** | fallback ناوبری /posts/new |
| `app/posts/[id]/edit/loading.tsx` | **جدید** | fallback ناوبری /posts/[id]/edit |
| `app/posts/page.tsx` | تغییر | لیست با خوانش کش‌شده + لینک «پست جدید» |
| `app/posts/[id]/page.tsx` | تغییر | جزئیات + لینک «ویرایش» + دکمهٔ حذف |
| `docs/server-actions-fa.md` | **جدید** | همین مقاله |
| `data/posts.json** | بدون تغییر | دیده‌بان seed |

---

## ۱۱. خروجی build و نقشهٔ مسیرها

### نمادها

| نماد | معنی |
|------|------|
| `○` (Static) | پوستهٔ استاتیک — بدون اجرای سرور |
| `◐` (Partial Prerender) | پوستهٔ استاتیک + محتوای پویای استریم‌شده |
| `ƒ` (Dynamic) | کاملاً سمت سرور رندر می‌شود |

### خروجی مورد انتظار

```
Route (app)
┌ ○ /
├ ○ /_not-found
├ ƒ /api/posts
├ ƒ /api/posts/[id]
├ ƒ /api/posts/popular
├ ◐ /posts              ← پوستهٔ استاتیک + لیست کش‌شده (استریم)
├ ◐ /posts/[id]         ← پوستهٔ استاتیک + جزئیات کش‌شده (استریم)
│ ├ /posts/[id]
│ ├ /posts/3
│ ├ /posts/1
│ └ [+3 more paths]     ← ۵ صفحهٔ پربازدید
├ ◐ /posts/[id]/edit    ← فرم ویرایش (استریم داده)
└ ○ /posts/new          ← فرم ثبت (بدون داده → استاتیک)
```

### چرا `/posts` و `/posts/[id]` از نوع `◐` هستند؟

چون `<Suspense>` دور خوانش کش‌شده را گرفته. Next.js پوستهٔ HTML را از قبل تولید می‌کند و محتوای `<Suspense>` را **استریم** می‌کند. این دقیقاً مدل PPR (Partial Prerender) است — پوستهٔ سریع + محتوای تازه.

---

## ۱۲. آزمایش و تأیید عملکرد

### ۱. Lint

```bash
npx eslint .
```

خروجی خالی = بدون خطا.

### ۲. Build

```bash
node node_modules/next/dist/bin/next build
```

> ⚠️ اگر API در دسترس نباشد، `generateStaticParams` با fallback `[]` اجرا می‌شود و صفحات پربازدید پیش‌رندر نمی‌شوند. build با موفقیت انجام می‌شود ولی صفحات `○` به `◐` تبدیل نمی‌شوند.

### ۳. آزمایش API با curl

<div dir="ltr">

```bash
# شروع سرور توسعه
node node_modules/next/dist/bin/next dev

# POST نامعتبر → 400
curl -s -X POST localhost:3000/api/posts \
  -H 'content-type: application/json' \
  -d '{"title":"t","content":"","author":"a"}'
# خروجی: {"errors":{"content":"محتوا نمی‌تواند خالی باشد."}}

# POST معتبر → 201
curl -s -w '\n%{http_code}\n' -X POST localhost:3000/api/posts \
  -H 'content-type: application/json' \
  -d '{"title":"پست جدید","content":"محتوا","author":"نویسنده"}'
# خروجی: {"id":11,"title":"پست جدید","slug":"post-11",...} 201

# PUT → 200
curl -s -X PUT localhost:3000/api/posts/11 \
  -H 'content-type: application/json' \
  -d '{"title":"ویرایش شده","content":"جدید","author":"نویسنده"}'
# خروجی: {"id":11,"title":"ویرایش شده",...} 200

# DELETE → 204
curl -s -o /dev/null -w '%{http_code}\n' \
  -X DELETE localhost:3000/api/posts/11
# خروجی: 204

# DELETE ناموجود → 404
curl -s -o /dev/null -w '%{http_code}\n' \
  -X DELETE localhost:3000/api/posts/999
# خروجی: 404
```

</div>

### ۴. آزمایش UI (دستی)

1. **ساخت:** برو به `/posts/new` → فیلدها را پر کن → دکمه «ثبت پست» → «در حال ذخیره…» → ریدایرکت به `/posts` → پست جدید **بلافاصله** در لیست (بدون رفرش)
2. **خطا:** فیلد عنوان را خالی بگذار → submit → خطا «عنوان الزامی است» زیر فیلد → متن تایپ‌شده حفظ شده
3. **ویرایش:** روی پست کلیک → «ویرایش» → اسکلتون ۱.۵ ثانیه‌ای → فرم با دادهٔ پر شده → ذخیره → صفحهٔ جزئیات تازه
4. **حذف:** «حذف پست» → confirm → «در حال حذف…» → ریدایرکت → پست از لیست حذف شده

### ۵. لاگ کش (اختیاری)

```bash
NEXT_PRIVATE_DEBUG_CACHE=1 node node_modules/next/dist/bin/next dev
```

در لاگ‌ها، hit/miss کش و `updateTag("posts")` invalidation قابل مشاهده است.

---

## ۱۳. مشکلات رایج و نکات مهم

### خطا: `Type 'Promise<FormState>' is not assignable to type 'void | Promise<void>'`

**علت:** `deletePostAction` مستقیماً به `<form action>` متصل است. فرم `action` prop انتظار `void | Promise<void>` دارد.

**راه‌حل:** نوع برگشتی اکشن را `Promise<void>` کنید و خطاها را `return` کنید (نه `return FormState`).

---

### خطا: `Uncached data was accessed outside of <Suspense>`

**علت:** تحت `cacheComponents`، وقتی کش miss شود، داده «uncached» است. Next.js اجازه نمی‌دهد دادهٔ uncached خارج از `<Suspense>` بلاک کند.

**راه‌حل:** داده‌گیری را در یک فرزند جداگانه بگذارید و آن را با `<Suspense>` احاطه کنید:

<div dir="ltr">

```tsx
// ❌ اشتباه
export default async function Page() {
  const data = await getPostCached(id);  // uncached → خطا!
  return <div>{data.title}</div>;
}

// ✅ درست
async function Content({ id }: { id: string }) {
  const data = await getPostCached(id);  // uncached اما داخل Suspense
  return <div>{data.title}</div>;
}

export default async function Page() {
  return (
    <Suspense fallback={<Skeleton />}>
      <Content id={id} />
    </Suspense>
  );
}
```

</div>

---

### خطا: pnpm `ERR_PNPM_IGNORED_BUILDS`

pnpm نسخه‌های جدید هنگام `pnpm run` وضعیت build scripts را بررسی می‌کنند. اگر `sharp` یا `unrs-resolver` در ignore-builds باشند، pnpm crash می‌کند.

**راه‌حل‌ها:**
1. `pnpm approve-builds` اجرا کنید و اسکریپت‌ها را تأیید کنید
2. مستقیماً با `node node_modules/next/dist/bin/next dev` اجرا کنید
3. `node_modules/.bin/eslint` را مستقیماً اجرا کنید

---

### نکته: `updateTag` فقط در Server Action کار می‌کند

اگر بخواهید در route handler یا webhook کش را باطل کنید، باید از `revalidateTag` استفاده کنید:

```ts
// route handler / webhook
import { revalidateTag } from "next/cache";
revalidateTag("posts");  // نه updateTag
```

---

### نکته: `use cache` return values باید serializable باشند

خروجی تابعی که `use cache` دارد باید بتواند به JSON تبدیل شود. آبجکت‌های ساده مثل `Post` مشکلی ندارند. اما توابع، تاریخ‌های `Date`، یا `Map`/`Set` مشکل‌ساز هستند.

---

## خلاصهٔ مفاهیم کلیدی

<div dir="ltr">

```
cacheComponents: true
        │
        ├── "use cache"         → خروجی را کش کن
        ├── cacheLife()         → عمر کش را تعیین کن
        ├── cacheTag()          → برچسب برای invalidation گروهی
        ├── updateTag()         → فقط Server Action: read-your-own-writes
        ├── revalidateTag()     → route/webhook: stale-while-revalidate
        └── revalidatePath()    → حذف کل مسیر از کش

"use client" کامپوننت‌ها:
        ├── useActionState()    → state فرم (errors) + action
        ├── useFormStatus()     → pending state → disable + label
        └── defaultValue        → حفظ متن پس از خطا

بارگذاری:
        ├── <Suspense>          → استریم محتوای پویا (الزامی تحت cacheComponents)
        ├── loading.tsx         → fallback ناوبری سطح مسیر
        └── FormSkeleton        → اسکلتون مشترک فرم‌ها
```

</div>

---

> پایان راهنما. این مقاله مرجع کدهای همین پروژه است و تمام مفاهیم عملی‌شده را پوشش می‌دهد.
</div>
