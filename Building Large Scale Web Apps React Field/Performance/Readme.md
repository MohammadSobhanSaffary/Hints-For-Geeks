# بهینه‌سازی عملکرد برنامه‌های وب

## مقدمه
این مستند شامل تکنیک‌ها و ابزارهای کلیدی برای بهینه‌سازی عملکرد برنامه‌های وب است، به‌ویژه برای برنامه‌های مدرن مانند برنامه‌های وب پیشرفته (PWA). این متن شامل توضیحات دقیق، نحوه کمک هر تکنیک به بهبود عملکرد، و مثال‌های کدنویسی است.

---

## فهرست مطالب
1. [Service Workers](#service-workers)
2. [Precaching](#precaching)
3. [Navigation Preload](#navigation-preload)
4. [Streaming HTML](#streaming-html)
5. [V8 Code Caching and Bytecode Cache](#v8-code-caching-and-bytecode-cache)
6. [Stale-While-Revalidate (SWR)](#stale-while-revalidate-swr)
7. [Brotli Compression](#brotli-compression)
8. [HTTP/3, Early Hints, and Priority Hints](#http3-early-hints-and-priority-hints)
9. [Client Hints and Device-Aware Content Delivery](#client-hints-and-device-aware-content-delivery)
10. [Adaptive Loading](#adaptive-loading)
11. [Virtual Lists and Infinite Scroll](#virtual-lists-and-infinite-scroll)
12. [Image Optimization](#image-optimization)

---

## 1. Service Workers
### توضیحات
**Service Worker** یک اسکریپت جاوااسکریپت است که در پس‌زمینه اجرا می‌شود و ویژگی‌هایی مانند کشینگ، اعلان‌های پوش و کنترل درخواست‌های شبکه را امکان‌پذیر می‌سازد. این اسکریپت دسترسی مستقیم به DOM ندارد.

### مزایا:
- **تجربه آفلاین:** ذخیره‌سازی منابع استاتیک برای دسترسی در حالت آفلاین.
- **کاهش تأخیر:** ارائه سریع منابع کش‌شده.
- **کنترل درخواست‌های شبکه:** پیاده‌سازی منطق سفارشی برای درخواست‌ها.

### مثال:
```javascript
// ثبت Service Worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('Service Worker ثبت شد', reg))
    .catch(err => console.error('ثبت Service Worker شکست خورد', err));
}

// فایل sw.js
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('v1').then(cache => {
      return cache.addAll([
        '/index.html',
        '/style.css',
        '/script.js',
        '/image.png'
      ]);
    })
  );
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response => {
      return response || fetch(event.request);
    })
  );
});
```

---

## 2. Precaching
### توضیحات
**Precaching** فرآیندی است که در آن منابع کلیدی در مرحله نصب Service Worker پیش‌بارگذاری می‌شوند.

### مزایا:
- **کاهش زمان بارگذاری:** منابع حیاتی از قبل کش می‌شوند.
- **تجربه بهتر آفلاین:** اطمینان از دسترسی همیشگی به منابع کلیدی.

### مثال:
```javascript
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('static-cache-v1').then(cache => {
      return cache.addAll([
        '/index.html',
        '/styles/main.css',
        '/scripts/app.js',
        '/images/logo.png'
      ]);
    })
  );
});
```

---

## 3. Navigation Preload
### توضیحات
**Navigation Preload** اجازه می‌دهد مرورگر درخواست‌های ناوبری را هم‌زمان با آماده‌سازی Service Worker پیش‌بارگذاری کند.

### مزایا:
- **کاهش تأخیر:** درخواست‌های ناوبری با فعال‌سازی Service Worker مسدود نمی‌شوند.
- **هماهنگی بهتر شبکه:** کش و درخواست شبکه به‌طور هم‌زمان انجام می‌شوند.

### مثال:
```javascript
self.addEventListener('activate', event => {
  event.waitUntil(self.registration.navigationPreload.enable());
});

self.addEventListener('fetch', event => {
  if (event.preloadResponse) {
    event.respondWith(
      event.preloadResponse.then(response => response || fetch(event.request))
    );
  }
});
```

---

## 4. Streaming HTML
### توضیحات
**Streaming HTML** شامل ارسال تکه‌های HTML از سرور به مرورگر به صورت تدریجی است که امکان رندر سریع‌تر را فراهم می‌کند.

### مزایا:
- **کاهش زمان تا اولین بایت (TTFB):** محتوای جزئی سریع‌تر رندر می‌شود.
- **پردازش هم‌زمان:** بارگذاری منابع هم‌زمان با دریافت محتوا.
- **تجربه بهتر در شبکه‌های کند:** کاربران محتوا را به‌صورت تدریجی مشاهده می‌کنند.

### مثال:
```javascript
const http = require('http');
http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.write('<!DOCTYPE html><html><head><title>Streaming</title></head><body>');
  res.write('<h1>خوش آمدید!</h1>');
  setTimeout(() => {
    res.write('<p>در حال بارگذاری محتوا...</p>');
    res.end('</body></html>');
  }, 2000);
}).listen(3000, () => console.log('سرور روی http://localhost:3000 اجرا شد'));
```

---

## 5. V8 Code Caching and Bytecode Cache
### توضیحات
- **Code Caching:** خروجی کامپایل جاوااسکریپت را برای استفاده سریع‌تر ذخیره می‌کند.
- **Bytecode Caching:** نسخه میانی کد (Bytecode) را برای اجرا ذخیره می‌کند.

### مزایا:
- **اجرای سریع‌تر:** نیاز به تجزیه و کامپایل مجدد حذف می‌شود.
- **کاهش استفاده از CPU:** بهینه‌سازی شده برای اسکریپت‌های پرکاربرد.

### مثال:
استفاده از DevTools برای پروفایلینگ اسکریپت‌های کش‌شده.

---

## 6. Stale-While-Revalidate (SWR)
### توضیحات
**SWR** داده‌های کش‌شده را فوراً ارائه می‌دهد و به‌طور هم‌زمان آن را در پس‌زمینه به‌روزرسانی می‌کند.

### مزایا:
- **پاسخ فوری:** داده‌های کش‌شده بلافاصله نمایش داده می‌شوند.
- **به‌روزرسانی در پس‌زمینه:** داده‌ها بدون تأخیر به‌روز می‌شوند.

### مثال:
```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(cachedResponse => {
      const fetchPromise = fetch(event.request).then(networkResponse => {
        caches.open('my-cache').then(cache => cache.put(event.request, networkResponse.clone()));
        return networkResponse;
      });
      return cachedResponse || fetchPromise;
    })
  );
});
```

---

## 7. Brotli Compression
### توضیحات
**Brotli** یک الگوریتم فشرده‌سازی مدرن است که توسط گوگل توسعه یافته و برای کاهش اندازه فایل‌های متنی مانند HTML، CSS و JavaScript طراحی شده است. این الگوریتم به عنوان جایگزینی بهینه‌تر برای Gzip شناخته می‌شود.

### مزایا:
- **فشرده‌سازی بهتر:** Brotli معمولاً فایل‌ها را 15-25٪ بیشتر از Gzip فشرده می‌کند.
- **سرعت بازگشایی بالا:** فرآیند باز کردن فشرده‌سازی (decompression) بسیار سریع است.
- **پشتیبانی مرورگرها:** تمامی مرورگرهای مدرن از Brotli پشتیبانی می‌کنند.

### مثال پیکربندی در Nginx:
```nginx
brotli on;
brotli_comp_level 6;
brotli_types text/html text/css text/javascript application/javascript application/json;
```

---

## 8. HTTP/3, Early Hints, and Priority Hints
### HTTP/3
### توضیحات
**HTTP/3** نسخه جدید پروتکل HTTP است که بر پایه QUIC ساخته شده و از UDP به جای TCP استفاده می‌کند. این پروتکل تأخیر را کاهش داده و عملکرد بهتری در شبکه‌های ناپایدار ارائه می‌دهد.

### مزایا:
- **اتصال سریع‌تر:** حذف نیاز به handshake‌های TCP.
- **مدیریت بهتر packet loss:** کاهش تأثیر از دست رفتن بسته‌ها در شبکه.
- **بدون انسداد خط مقدم (Head-of-Line Blocking):** جریان‌های مستقل داده‌ها.

### Early Hints (103)
### توضیحات
کد وضعیت HTTP **103 Early Hints** به مرورگر اجازه می‌دهد قبل از دریافت پاسخ نهایی، منابع حیاتی مانند CSS و JavaScript را پیش‌بارگذاری کند.

### مزایا:
- **کاهش زمان بارگذاری:** مرورگر می‌تواند منابع را زودتر دانلود کند.
- **بهبود تجربه کاربری:** کاهش زمان انتظار برای کاربران.

### Priority Hints
### توضیحات
**Priority Hints** امکان تعیین اولویت بارگذاری منابع را فراهم می‌کند.

### مزایا:
- **بهینه‌سازی پهنای باند:** بارگذاری منابع حیاتی سریع‌تر انجام می‌شود.
- **کنترل بهتر توسعه‌دهندگان:** اولویت‌دهی به محتوای مهم‌تر.

### مثال Priority Hints:
```html
<img src="hero.jpg" importance="high" alt="تصویر اصلی">
<img src="decorative.jpg" importance="low" alt="تصویر تزئینی">
```

---

## 9. Client Hints and Device-Aware Content Delivery
### توضیحات
**Client Hints** مجموعه‌ای از هدرهای HTTP است که اطلاعاتی مانند اندازه صفحه، نوع دستگاه، و کیفیت شبکه را به سرور ارسال می‌کند. **Device-Aware Content Delivery** به ارائه محتوای متناسب با دستگاه کاربر کمک می‌کند.

### مزایا:
- **محتوای تطبیقی:** ارسال نسخه مناسب محتوا بر اساس دستگاه.
- **کاهش حجم داده‌ها:** ارسال تصاویر و ویدئوهای فشرده برای دستگاه‌های ضعیف یا شبکه‌های کند.

### مثال Client Hints:
```http
Sec-CH-Width: 1080
Sec-CH-DPR: 2.0
Sec-CH-Viewport-Width: 375
Sec-CH-ECT: 4g
```

---

## 10. Adaptive Loading
### توضیحات
**Adaptive Loading** روشی برای بهینه‌سازی بارگذاری منابع بر اساس شرایط دستگاه و شبکه کاربر است. این روش از اطلاعاتی مانند نوع دستگاه، حافظه، و کیفیت شبکه برای ارائه تجربه بهینه استفاده می‌کند.

### مزایا:
- **تجربه بهتر در دستگاه‌های ضعیف:** کاهش بار روی دستگاه‌های کم‌توان.
- **بهبود عملکرد در شبکه‌های کند:** ارائه نسخه‌های سبک‌تر محتوا.

### مثال در React:
```javascript
import { useEffect, useState } from 'react';

const useDeviceInfo = () => {
  const [deviceInfo, setDeviceInfo] = useState({
    effectiveConnectionType: '4g',
   



## 11. Virtual Lists and Infinite Scroll
### توضیحات
**Virtual Lists** تکنیکی برای رندر کردن تنها آیتم‌های قابل مشاهده در لیست‌های بزرگ است. **Infinite Scroll** داده‌ها را هنگام اسکرول به پایین صفحه به صورت پویا بارگذاری می‌کند.

### مزایا:
- **کاهش بار پردازشی:** رندر آیتم‌های کمتر در هر لحظه.
- **بارگذاری تدریجی داده‌ها:** بهبود عملکرد و کاهش مصرف منابع.

### مثال با React Virtualized:
```javascript
import { List } from 'react-virtualized';

function VirtualizedList({ items }) {
  return (
    <List
      width={300}
      height={600}
      rowCount={items.length}
      rowHeight={50}
      rowRenderer={({ index, key, style }) => (
        <div key={key} style={style}>
          {items[index]}
        </div>
      )}
    />
  );
}
```

---

## 12. Image Optimization
### توضیحات
بهینه‌سازی تصاویر شامل کاهش اندازه فایل تصاویر بدون کاهش کیفیت آن‌ها است.

### مزایا:
- **کاهش زمان بارگذاری:** تصاویر سبک‌تر سریع‌تر بارگذاری می‌شوند.
- **صرفه‌جویی در پهنای باند:** استفاده از تصاویر فشرده‌تر.

### ابزارهای پیشنهادی:
- **ImageMagick:** ابزار خط فرمان برای بهینه‌سازی تصاویر.
- **TinyPNG:** سرویس آنلاین برای فشرده‌سازی تصاویر PNG و JPEG.
- **WebP:** فرمت مدرن با اندازه کمتر و کیفیت بهتر.

### مثال استفاده از WebP:
```html
<picture>
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="توضیحات تصویر">
</picture>
```

