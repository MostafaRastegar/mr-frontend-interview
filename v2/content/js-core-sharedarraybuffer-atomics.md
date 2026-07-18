<div dir="rtl">

‏شما یک متخصص ارشد فرانت‌اند هستید که در حال آماده‌سازی برای مصاحبه فنی می‌باشد.

‏لطفاً سوال "SharedArrayBuffer و Atomics برای محاسبات موازی در Web Workers" را بر اساس یک چارچوب تحلیلی ۴ سطحی برای من تشریح کن. سطح پاسخ باید مختص یک توسعه‌دهنده Senior با ۶+ سال سابقه باشد که در پروژه‌های بزرگ مقیاس (Enterprise) کار کرده است.

</div>

---

## <div dir="rtl">‏سطح ۱: تعریف و هسته فنی</div>

<div dir="rtl">

‏**SharedArrayBuffer** — یک بافر حافظه‌ی خام (raw bytes) که بین main thread و Web Workers به اشتراک گذاشته می‌شود. برخلاف `postMessage` که داده را کپی می‌کند، SAB حافظه را مستقیماً در معرض view قرار می‌دهد — صفر کپی.

‏**Atomics** — مجموعه‌ای از عملیات atomic (غیرقابل interrupt) روی SharedArrayBuffer: خواندن، نوشتن، compare-and-swap، wait/notify (همانند futex در Linux).

</div>

```javascript
// Main thread
const sab = new SharedArrayBuffer(4 * 1024); // 4KB shared memory
const view = new Int32Array(sab);
const worker = new Worker('worker.js');

worker.postMessage(sab); // انتقال ownership (بدون کپی)

// Worker
self.onmessage = (e) => {
  const sab = e.data;
  const view = new Int32Array(sab);

  // Atomic write
  Atomics.store(view, 0, 42);
  // Atomic read
  const val = Atomics.load(view, 0);
  // Atomic compare-and-swap
  const old = Atomics.compareExchange(view, 0, 42, 100);
  // Wait for notification
  Atomics.wait(view, 1, 0); // blocked until another thread notifies
};

// Main: notify worker
Atomics.store(view, 1, 1);
Atomics.notify(view, 1, 1); // wake up one waiter
```

<div dir="rtl">

‏**محدودیت امنیتی:** SharedArrayBuffer به `Cross-Origin-Opener-Policy: same-origin` و `Cross-Origin-Embedder-Policy: require-corp` نیاز دارد (SAB after Spectre).

</div>

---

## <div dir="rtl">‏سطح ۲: معیارهای انتخاب و تصمیم‌گیری</div>

<div dir="rtl">

‏| رویکرد | موارد استفاده | مزیت | محدودیت |
‏|--------|--------------|------|---------|
‏| `postMessage` copy | کارهای مجزا (هر worker مستقل) | ساده, امن | overhead کپی برای داده‌ی بزرگ |
‏| SharedArrayBuffer | داده‌ی مشترک بزرگ (video frame, audio buffer, game state) | صفر کپی, real-time | پیچیدگی synchronisation |
‏| `Atomics.wait/notify` | Producer-consumer درون workers | بدون busy-wait | blocking در worker, کوئ امن نیست |
‏| Transferable (`{transfer: ...}`) | انتقال ownership (ArrayBuffer, ImageBitmap) | صفر کپی, یک‌بار مصرف | بعد از انتقال در sender undefined می‌شود |

</div>

```javascript
// x Video processing pipeline با SAB
class VideoProcessor {
  constructor(workerCount = 4) {
    this.frameSize = 1920 * 1080 * 4; // RGBA
    this.sab = new SharedArrayBuffer(this.frameSize);
    this.workers = [];

    for (let i = 0; i < workerCount; i++) {
      const w = new Worker('frame-worker.js');
      w.postMessage({ sab: this.sab, start: i * this.frameSize / workerCount, end: (i+1) * this.frameSize / workerCount });
      this.workers.push(w);
    }
  }

  processFrame(frameData) {
    // Copy frame to SAB
    new Uint8Array(this.sab).set(new Uint8Array(frameData));

    // Notify all workers to start processing
    Atomics.store(new Int32Array(this.sab, 0, 1), 0, 1);
    Atomics.notify(new Int32Array(this.sab, 0, 1), 0, this.workers.length);

    // Wait for all to complete
    for (let i = 0; i < this.workers.length; i++) {
      Atomics.wait(new Int32Array(this.sab, 4 + i*4, 1), 0, 0);
    }

    return new Uint8Array(this.sab).slice();
  }
}
```

---

## <div dir="rtl">‏سطح ۳: چالش واقعی در پروژه‌های بزرگ</div>

<div dir="rtl">

‏**سناریو:** یک dashboard real-time با ۱۰۰+ نماد سهام — هر symbol قیمت لحظه‌ای از WebSocket می‌آید. WebSocket handler در main thread است و ۴ worker شاخص‌های تکنیکال (SMA, RSI, MACD) را محاسبه می‌کنند.

‏**چالش:** اگر همه‌ی symbolها در یک SAB بنویسند و workerها بخوانند، **data race** رخ می‌دهد — worker نصف قیمت قدیم و نصف قیمت جدید را می‌خواند.

‏**راه‌حل:** Double buffering + Atomics برای safe handover.

</div>

```javascript
// Double-buffer SAB
class RingBuffer {
  constructor(slots, slotSize) {
    this.sab = new SharedArrayBuffer(slots * slotSize + 8); // 8 bytes for metadata
    this.meta = new Int32Array(this.sab, 0, 2);
    this.slots = slots;
    this.slotSize = slotSize;
  }

  write(data, slotIndex) {
    const offset = 8 + (slotIndex % this.slots) * this.slotSize;
    const target = new Uint8Array(this.sab, offset, this.slotSize);
    target.set(new Uint8Array(data));

    // x Publish: update version بعد از نوشتن کامل
    Atomics.add(this.meta, 0, 1); // write version
    Atomics.store(this.meta, 1, slotIndex); // current slot
    Atomics.notify(this.meta, 1, 1); // wake consumer
  }

  read(slotIndex, lastVersion) {
    const meta = new Int32Array(this.sab, 0, 2);
    const currentVersion = Atomics.load(meta, 0);

    // x اگر نسخه تغییر نکرده، داده همان است (no new data)
    if (currentVersion === lastVersion) return null;

    const offset = 8 + (slotIndex % this.slots) * this.slotSize;
    const data = new Uint8Array(this.sab, offset, this.slotSize);
    return { data: data.slice(), version: currentVersion };
  }
}
```

<div dir="rtl">

‏**نکته Production:** `Atomics.wait` worker thread را **block** می‌کند — برخلاف `postMessage`. در worker از `Atomics.waitAsync` (غیرمسدودکننده) استفاده کن تا worker بتواند eventهای دیگر را هم پردازش کند.

</div>

---

## <div dir="rtl">‏سطح ۴: عدم استفاده و راه‌کار جایگزین</div>

<div dir="rtl">

‏بدون SharedArrayBuffer:

</div>

| راهکار | توضیح | محدودیت |
|--------|-------|---------|
| `postMessage` + transferable | کپی ownership | یک‌بار مصرف, هزینه allocation برای هر فریم |
| Canvas + `OffscreenCanvas` | رندر در worker + transfer | فقط graphics, نه general data |
| WebRTC DataChannel | P2P بین workers | overkill, latency |
| Service Worker + Cache API | اشتراک data از طریق cache | async, latency بالا |

<div dir="rtl">

‏**مقایسه Next.js vs Express.js:** در Next.js Server Components (Node.js) می‌توانی از `worker_threads` با `SharedArrayBuffer` استفاده کنی — اما در Edge Runtime **SharedArrayBuffer در دسترس نیست**. در Express.js (Node.js) محدودیت مشابهی داری — `worker_threads` برای CPU-intensive tasks. Next.js App Router با Server Components به طور طبیعی computation-heavy منطق را سمت سرور می‌برد، بنابراین نیاز به worker در مرورگر کمتر می‌شود.

</div>

<div dir="rtl">

‏> **نهایی:** SharedArrayBuffer ابزار سطح پایین برای worker communication بدون کپی است. برای ۹۰٪ موارد، `postMessage` کافی است. SAB را فقط زمانی استفاده کن که latency single-digit میلی‌ثانیه نیاز داری (audio, video, game). مهم‌ترین نکته: همیشه از `Atomics` برای دسترسی استفاده کن — دسترسی مستقیم (`view[0] = x`) در V8 ممکن است race condition ایجاد کند چون compiler می‌تواند دستورات را مرتب کند.

</div>
