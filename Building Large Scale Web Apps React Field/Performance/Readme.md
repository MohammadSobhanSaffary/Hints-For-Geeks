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

### منابع برای مطالعه بیشتر:
- [MDN: Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)
- [web.dev: Learn Service Workers](https://web.dev/learn/pwa/service-workers/)

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

### منابع برای مطالعه بیشتر:
- [MDN: Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)
- [web.dev: Precaching with Workbox](https://web.dev/workbox-precaching/)

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

### منابع برای مطالعه بیشتر:
- [web.dev: Navigation Preload](https://web.dev/navigation-preload/)

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

### منابع برای مطالعه بیشتر:
- [MDN: Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

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

### منابع برای مطالعه بیشتر:
- [V8: Understanding Caching](https://v8.dev/blog/code-caching)

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

### منابع برای مطالعه بیشتر:
- [MDN: Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)

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

### منابع برای مطالعه بیشتر:
- [MDN: Content-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding)
- [web.dev: Brotli Compression](https://web.dev/uses-text-compression/)

---

## 8. HTTP/3, Early Hints, and Priority Hints
### منابع:
- [HTTP/3 Overview (web.dev)](https://web.dev/http3/)
- [Priority Hints (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/importance)

(ادامه مستند با افزودن منابع تکمیل می‌شود...)


بله، حتما! در ادامه، لینک‌های منابع مطالعه برای بخش‌های ۸ تا ۱۲ به مستند اضافه می‌شود:

---


### HTTP/3
**توضیحات:**  
**HTTP/3** نسخه جدید پروتکل HTTP است که بر پایه QUIC ساخته شده و از UDP به جای TCP استفاده می‌کند. این پروتکل تأخیر را کاهش داده و عملکرد بهتری در شبکه‌های ناپایدار ارائه می‌دهد.

**مزایا:**
- **اتصال سریع‌تر:** حذف نیاز به handshake‌های TCP.
- **مدیریت بهتر packet loss:** کاهش تأثیر از دست رفتن بسته‌ها در شبکه.
- **بدون انسداد خط مقدم (Head-of-Line Blocking):** جریان‌های مستقل داده‌ها.

**منابع برای مطالعه بیشتر:**
- [HTTP/3 Overview (web.dev)](https://web.dev/http3/)

### Early Hints (103)
**توضیحات:**  
کد وضعیت HTTP **103 Early Hints** به مرورگر اجازه می‌دهد قبل از دریافت پاسخ نهایی، منابع حیاتی مانند CSS و JavaScript را پیش‌بارگذاری کند.

**مزایا:**
- **کاهش زمان بارگذاری:** مرورگر می‌تواند منابع را زودتر دانلود کند.
- **بهبود تجربه کاربری:** کاهش زمان انتظار برای کاربران.

**منابع برای مطالعه بیشتر:**
- [Early Hints (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/103)

### Priority Hints
**توضیحات:**  
**Priority Hints** امکان تعیین اولویت بارگذاری منابع را فراهم می‌کند.

**مزایا:**
- **بهینه‌سازی پهنای باند:** بارگذاری منابع حیاتی سریع‌تر انجام می‌شود.
- **کنترل بهتر توسعه‌دهندگان:** اولویت‌دهی به محتوای مهم‌تر.

**مثال Priority Hints:**
```html
<img src="hero.jpg" importance="high" alt="تصویر اصلی">
<img src="decorative.jpg" importance="low" alt="تصویر تزئینی">
```

**منابع برای مطالعه بیشتر:**
- [Priority Hints (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/importance)

---

## 9. Client Hints and Device-Aware Content Delivery

**توضیحات:**  
**Client Hints** مجموعه‌ای از هدرهای HTTP است که اطلاعاتی مانند اندازه صفحه، نوع دستگاه، و کیفیت شبکه را به سرور ارسال می‌کند. **Device-Aware Content Delivery** به ارائه محتوای متناسب با دستگاه کاربر کمک می‌کند.

**مزایا:**
- **محتوای تطبیقی:** ارسال نسخه مناسب محتوا بر اساس دستگاه.
- **کاهش حجم داده‌ها:** ارسال تصاویر و ویدئوهای فشرده برای دستگاه‌های ضعیف یا شبکه‌های کند.

**مثال Client Hints:**
```http
Sec-CH-Width: 1080
Sec-CH-DPR: 2.0
Sec-CH-Viewport-Width: 375
Sec-CH-ECT: 4g
```

**منابع برای مطالعه بیشتر:**
- [Client Hints (web.dev)](https://web.dev/client-hints/)
- [Client Hints (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Client-Hints)

---

## 10. Adaptive Loading

**توضیحات:**  
**Adaptive Loading** روشی برای بهینه‌سازی بارگذاری منابع بر اساس شرایط دستگاه و شبکه کاربر است. این روش از اطلاعاتی مانند نوع دستگاه، حافظه، و کیفیت شبکه برای ارائه تجربه بهینه استفاده می‌کند.

**مزایا:**
- **تجربه بهتر در دستگاه‌های ضعیف:** کاهش بار روی دستگاه‌های کم‌توان.
- **بهبود عملکرد در شبکه‌های کند:** ارائه نسخه‌های سبک‌تر محتوا.

**منابع برای مطالعه بیشتر:**
- [Adaptive Loading (web.dev)](https://web.dev/adaptive-loading/)
- [Responsive Images (MDN)](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)

---

## 11. Virtual Lists and Infinite Scroll

**توضیحات:**  
**Virtual Lists** تکنیکی برای رندر کردن تنها آیتم‌های قابل مشاهده در لیست‌های بزرگ است. **Infinite Scroll** داده‌ها را هنگام اسکرول به پایین صفحه به صورت پویا بارگذاری می‌کند.

**مزایا:**
- **کاهش بار پردازشی:** رندر آیتم‌های کمتر در هر لحظه.
- **بارگذاری تدریجی داده‌ها:** بهبود عملکرد و کاهش مصرف منابع.

**مثال با React Virtualized:**
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

**منابع برای مطالعه بیشتر:**
- [Virtualization (patterns.dev)](https://www.patterns.dev/vanilla/virtual-lists/)
- [Virtualized Lists (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)

---

## 12. Image Optimization

**توضیحات:**  
بهینه‌سازی تصاویر شامل کاهش اندازه فایل تصاویر بدون کاهش کیفیت آن‌ها است.

**مزایا:**
- **کاهش زمان بارگذاری:** تصاویر سبک‌تر سریع‌تر بارگذاری می‌شوند.
- **صرفه‌جویی در پهنای باند:** استفاده از تصاویر فشرده‌تر.

**ابزارهای پیشنهادی:**
- **ImageMagick:** ابزار خط فرمان برای بهینه‌سازی تصاویر.
- **TinyPNG:** سرویس آنلاین برای فشرده‌سازی تصاویر PNG و JPEG.
- **WebP:** فرمت مدرن با اندازه کمتر و کیفیت بهتر.

**مثال استفاده از WebP:**
```html
<picture>
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="توضیحات تصویر">
</picture>
```

**منابع برای مطالعه بیشتر:**
- [Image Optimization (web.dev)](https://web.dev/optimize-images/)
- [Image Formats (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture)











# Optimizing Web Application Performance

## Introduction  
This document covers key techniques and tools for optimizing the performance of web applications, particularly modern applications like Progressive Web Apps (PWA). It includes detailed explanations, how each technique helps improve performance, and coding examples.

---

## Table of Contents  
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
### Explanation  
A **Service Worker** is a JavaScript script that runs in the background, enabling features like caching, push notifications, and network request control. It has no direct access to the DOM.

### Benefits:  
- **Offline Experience:** Caching static resources for offline access.  
- **Reduced Latency:** Quickly serving cached resources.  
- **Network Request Control:** Implement custom logic for requests.  

### Example:  
```javascript
// Registering a Service Worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('Service Worker registered', reg))
    .catch(err => console.error('Service Worker registration failed', err));
}

// sw.js file
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

### Further Reading:  
- [MDN: Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)  
- [web.dev: Learn Service Workers](https://web.dev/learn/pwa/service-workers/)

---

## 2. Precaching  
### Explanation  
**Precaching** is the process of preloading key resources during the Service Worker installation phase.

### Benefits:  
- **Faster Load Times:** Critical resources are cached beforehand.  
- **Better Offline Experience:** Ensures key resources are always available.  

### Example:  
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

### Further Reading:  
- [MDN: Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)  
- [web.dev: Precaching with Workbox](https://web.dev/workbox-precaching/)

---

## 3. Navigation Preload  
### Explanation  
**Navigation Preload** allows the browser to preload navigation requests while the Service Worker is being prepared.

### Benefits:  
- **Reduced Latency:** Navigation requests are not blocked by Service Worker activation.  
- **Better Network Coordination:** Cache and network requests are handled simultaneously.  

### Example:  
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

### Further Reading:  
- [web.dev: Navigation Preload](https://web.dev/navigation-preload/)

---

## 4. Streaming HTML  
### Explanation  
**Streaming HTML** involves sending chunks of HTML from the server to the browser gradually, allowing for faster rendering.

### Benefits:  
- **Reduced Time to First Byte (TTFB):** Partial content is rendered quicker.  
- **Concurrent Processing:** Resources are loaded simultaneously with content retrieval.  
- **Better Experience on Slow Networks:** Users see content gradually as it loads.  

### Example:  
```javascript
const http = require('http');
http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.write('<!DOCTYPE html><html><head><title>Streaming</title></head><body>');
  res.write('<h1>Welcome!</h1>');
  setTimeout(() => {
    res.write('<p>Content is loading...</p>');
    res.end('</body></html>');
  }, 2000);
}).listen(3000, () => console.log('Server running at http://localhost:3000'));
```

### Further Reading:  
- [MDN: Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

---

## 5. V8 Code Caching and Bytecode Cache  
### Explanation  
- **Code Caching:** Caches the output of JavaScript compilation for faster subsequent executions.  
- **Bytecode Caching:** Caches the intermediate bytecode for execution.

### Benefits:  
- **Faster Execution:** Eliminates the need for re-parsing and recompiling.  
- **Reduced CPU Usage:** Optimized for high-use scripts.  

### Example:  
Use DevTools to profile cached scripts.

### Further Reading:  
- [V8: Understanding Caching](https://v8.dev/blog/code-caching)

---

## 6. Stale-While-Revalidate (SWR)  
### Explanation  
**SWR** serves cached data immediately and simultaneously updates it in the background.

### Benefits:  
- **Immediate Response:** Cached data is shown instantly.  
- **Background Updates:** Data is refreshed without any delay.  

### Example:  
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

### Further Reading:  
- [MDN: Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)

---

## 7. Brotli Compression  
### Explanation  
**Brotli** is a modern compression algorithm developed by Google, designed for reducing the size of textual files like HTML, CSS, and JavaScript. It is considered a more efficient alternative to Gzip.

### Benefits:  
- **Better Compression:** Brotli compresses files 15-25% better than Gzip.  
- **Fast Decompression:** The decompression process is very fast.  
- **Browser Support:** All modern browsers support Brotli.  

### Example Nginx Configuration:  
```nginx
brotli on;
brotli_comp_level 6;
brotli_types text/html text/css text/javascript application/javascript application/json;
```

### Further Reading:  
- [MDN: Content-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding)  
- [web.dev: Brotli Compression](https://web.dev/uses-text-compression/)

---

## 8. HTTP/3, Early Hints, and Priority Hints  

### HTTP/3  
**Explanation:**  
**HTTP/3** is the latest version of the HTTP protocol built on top of QUIC and uses UDP instead of TCP. This protocol reduces latency and performs better on unstable networks.

**Benefits:**  
- **Faster Connections:** Eliminates the need for TCP handshakes.  
- **Better Packet Loss Management:** Reduces the impact of lost packets in the network.  
- **No Head-of-Line Blocking:** Independent data streams.  

### Early Hints (103)  
**Explanation:**  
HTTP status code **103 Early Hints** allows the browser to preload critical resources like CSS and JavaScript before receiving the final response.

**Benefits:**  
- **Reduced Load Time:** The browser can download resources earlier.  
- **Improved User Experience:** Reduces the waiting time for users.

### Priority Hints  
**Explanation:**  
**Priority Hints** allows developers to set loading priorities for resources.

**Benefits:**  
- **Optimized Bandwidth:** Critical resources are loaded faster.  
- **Better Developer Control:** Prioritize important content.

**Priority Hints Example:**  
```html
<img src="hero.jpg" importance="high" alt="Main image">
<img src="decorative.jpg" importance="low" alt="Decorative image">
```

### Further Reading:  
- [HTTP/3 Overview (web.dev)](https://web.dev/http3/)  
- [Priority Hints (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/importance)

---

## 9. Client Hints and Device-Aware Content Delivery  

### Explanation  
**Client Hints** is a set of HTTP headers that send device information such as screen size, device type, and network conditions to the server. **Device-Aware Content Delivery** helps deliver content tailored for the user’s device.

### Benefits:  
- **Adaptive Content:** Deliver appropriate content based on the device.  
- **Reduced Data Usage:** Send compressed images and videos for weak devices or slow networks.  

**Client Hints Example:**  
```http
Sec-CH-Width: 1080  
Sec-CH-DPR: 2.0  
Sec-CH-Viewport-Width: 375  
Sec-CH-ECT: 4g  
```

### Further Reading:  
- [Client Hints (web.dev)](https://web.dev/client-hints/)  
- [Client Hints (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Client-Hints)

---

## 10. Adaptive Loading  

### Explanation  
**Adaptive Loading** is a method to optimize resource loading based on the user's device and network conditions. It uses information like device type, memory, and network quality to provide an optimal experience.

### Benefits:  
- **Better Experience on Weak Devices:** Reduces load on less powerful devices.  
- **Improved Performance on Slow Networks:** Delivers lighter content versions.

### Further Reading:  
- [Adaptive Loading (web.dev)](https://web.dev/adaptive-loading/)  
- [Responsive Images (MDN)](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)

---

## 11. Virtual Lists and Infinite

 Scroll  

### Explanation  
**Virtual Lists** is a technique for rendering only the visible items in large lists. **Infinite Scroll** dynamically loads data as the user scrolls down the page.

### Benefits:  
- **Reduced Rendering Load:** Render fewer items at a time.  
- **Progressive Data Loading:** Load data on demand, improving resource usage.

**React Virtualized Example:**  
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

### Further Reading:  
- [Virtualization (web.dev)](https://web.dev/virtualization/)  
- [Virtualized Lists (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)

---

## 12. Image Optimization  

### Explanation  
**Image Optimization** includes reducing the file size of images without compromising quality.

### Benefits:  
- **Faster Load Times:** Lighter images load faster.  
- **Bandwidth Savings:** Smaller image files use less data.

**WebP Example:**  
```html
<picture>
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Image description">
</picture>
```

### Further Reading:  
- [Image Optimization (web.dev)](https://web.dev/optimize-images/)  
- [Image Formats (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture)

