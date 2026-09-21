# Aether Desktop 1.2.6

سه فیکس ریشه‌ای: ترجیح هستهٔ فیزیکی برای سطح کارایی، لاگ قابل‌فهم برای خاموش شدن DPAPI، و حذف شکست preflight.

## ۱) سطح کارایی — آستانهٔ ۱۲ هستهٔ منطقی

### مشکل
`detected_cpus()` از `std::thread::available_parallelism()` استفاده می‌کرد که روی ویندوز **هسته‌های منطقی** (hyperthreaded) برمی‌گرداند. یک لپ‌تاپ ۸ هسته‌ای با HT (مثلاً Intel Core i7-1270P = ۱۲ فیزیکی/۱۶ منطقی) یا هر ۸ هسته‌ای/۱۶ منطقی با `--perf high` مجبور می‌شد بافرهای ۱۶ مگابایتی/۳۲ مگابایتی باز کند — برای هیچ فایده‌ای.

### فیکس
آستانه از `>= 8` به `>= 12` منطقی تغییر یافت. با هایپرتریدینگ، ۱۲ منطقی ≈ ۶ فیزیکی — تضمین می‌کند فقط ماشین‌های واقعی سنگین بارِ بزرگ را بگیرند. لپ‌تاپ‌های خانگی ۸ هسته‌ای حالا هسته را ترجیح خودشان می‌دهند.

### عملکرد
- `cpus < 12` → هستهٔ خودش انتخاب می‌کند (معمولاً Medium)
- `cpus >= 12` → `high` تضمین می‌شود

## ۲) لاگ DPAPI — دلیل خاموش شدن کلید

### مشکل
وقتی `CryptUnprotectData` با `CRYPTPROTECT_UI_FORBIDDEN` شکست می‌خورد (مثلاً بعد از بازنشانی ویندوز، کپی `secrets.bin` به ماشین دیگر، یا خرابی پروفایل کاربر)، خطای opaque «`CryptUnprotectData failed: ...`» به لاگ اضافه می‌شد و کلید خاموش می‌شد. کاربر نمی‌دانست چرا و وارد کردن دوباره کلیدش از همان صفحه شروع بود.

### فیکس
خطای now پیام واضح می‌دهد: **«DPAPI failed to decrypt secrets: the key on disk is unrecoverable — this happens after a Windows reinstall or moving secrets.bin to another machine. Re-enter your Gemini key.»**

## ۳) core-rollback.ps1 در preflight

### مشکل
`scripts/core-rollback.ps1` در preflight required file list نبود. اگر `sync-core.sh` در بیلد با خطای rollback مواجه شود و این فایل missing باشد، بیلد بعدی بدون rollback path شکست می‌خورد.

### فیکس
فایل به لیست فایل‌های required اضافه شد — preflight حالا قبل از هر بیلد وجود آن را بررسی می‌کند.