+++
date = '2025-05-11T20:16:41+03:30'
draft = false
title = 'دانلود فایل‌های ترتیبی با ابزارهای gnu'
summary= "در این پست به این می‌پردازیم که چطور فایل‌های ترتیبی مثل بازی یا سریال رو در یک مرحله با wget دانلود کنیم."
tags= ["unix", "wget", "tr", "seq", "linux", "brace_expantion"]
+++

## مقدمه 

<figure>
    <img width="80%" alt="لینک‌های دانلود فصل ۵ یک سریال" title="لینک‌های دانلود فصل ۵ یک سریال" src="/images/better-call-saul-05-download-links.webp" ></img>
    <figcaption>
        نمونه‌ای از لینک‌های دانلود یک سریال؛ لینک‌ها فقط در یک عدد تفاوت دارند.
    </figcaption>
</figure>

سریال یا بازی که می‌خوام دانلود کنم باید چندین فایل پشت‌سر هم و به ترتیب دانلود بشه. الگوی لینک دانلودشون هم مشابه همه و فقط در یک شماره که ترتیب رو مشخص می‌کنه با هم تفاوت دارن. مثلاً:

```bash
https://example.com/Better.Call.Saul.S05E01.mkv
https://example.com/Better.Call.Saul.S05E02.mkv
https://example.com/Better.Call.Saul.S05E03.mkv
...
https://example.com/Better.Call.Saul.S05E10.mkv
```

می‌خوام خیلی سریع و ساده همهٔ فایل‌ها رو دانلود کنم. راه حل اصولیش اینه که بشینیم یک اسکریپت بنویسیم که url بگیره، صفحه رو scrape، لینک‌های دانلود رو استخراج و دانلود کنه. قطعاً کلی ابزار آماده هم برای این کار وجود داره. ولی از اونجایی که حوصلهٔ اسکریپت‌نویسی و تمایل استفاده از ابزارهای دیگه رو ندارم؛ دوست دارم با ابزارهای سادهٔ گنو این کار رو انجام بدم. تنها کاری که باید بکنیم اینه که تمامی لینک‌ها رو به ترتیب شمارشون تولید کنیم. بعدش می‌شه با ابزاری مثل `wget` به راحتی دانلودشون کرد.

## ایدهٔ اولیه: استفاده از brace expantion

اولین چیزی که به ذهنم رسید استفاده از *brace expantion* برای تولید لینک‌ها بود (چون شمارهٔ لینک‌ها zero pad هستن `01` گذاشتم. همچنین از اونجایی که `echo` از کاراکتر فاصله برای جدا کردن خروجی استفاده می‌کنه، یک پردازش نهایی با `tr` هم لازمه):

```bash
> echo https://example.com/Better.Call.Saul.S05E{01..10}.mkv | tr ' ' "\n"
https://example.com/Better.Call.Saul.S05E01.mkv
https://example.com/Better.Call.Saul.S05E02.mkv
https://example.com/Better.Call.Saul.S05E03.mkv
https://example.com/Better.Call.Saul.S05E04.mkv
https://example.com/Better.Call.Saul.S05E05.mkv
https://example.com/Better.Call.Saul.S05E06.mkv
https://example.com/Better.Call.Saul.S05E07.mkv
https://example.com/Better.Call.Saul.S05E08.mkv
https://example.com/Better.Call.Saul.S05E09.mkv
https://example.com/Better.Call.Saul.S05E10.mkv
```

حالا که تونستیم لینک‌ها رو تولید کنیم کافیه با `wget` دانلودشون کنیم. بسته به سرور، امکان داره که خطای `403` بگیریم. برای حل این مشکل کافیه یک `user-agent` برای `wget` تنظیم کنیم.

```bash
> wget -P ~/Downloads --user-agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36' https://example.com/Better.Call.Saul.S05E{01..10}.mkv | tr ' ' "\n"
```


کار ما با brace expantion به خوبی راه می‌افته ولی حین انجام این‌کار با دستور `seq` هم آشنا شدم که دونستن‌اش خالی از لطف نیست.

## روش دوم: `seq`

با دستور `seq` می‌تونیم دنباله‌ای از اعداد تولید کنیم. ویژگی جالبش آرگومان  `-w` هست که خروجی رو zero pad می‌کنه.

```bash
> seq -w 1 10
01
02
03
04
05
06
07
08
09
10
```

حالا کافیه `for` بزنیم و باقی آدرس لینک دانلود رو بهش بچسبونیم:

```bash
for i in $(seq -w 1 10); do
    echo "https://example.com/Better.Call.Saul.S05E$i.mkv"
done

https://example.com/Better.Call.Saul.S05E01.mkv
https://example.com/Better.Call.Saul.S05E02.mkv
https://example.com/Better.Call.Saul.S05E03.mkv
https://example.com/Better.Call.Saul.S05E04.mkv
https://example.com/Better.Call.Saul.S05E05.mkv
https://example.com/Better.Call.Saul.S05E06.mkv
https://example.com/Better.Call.Saul.S05E07.mkv
https://example.com/Better.Call.Saul.S05E08.mkv
https://example.com/Better.Call.Saul.S05E09.mkv
https://example.com/Better.Call.Saul.S05E10.mkv
```

حالا که تونستیم لینک‌ها رو به درستی تولید کنیم، کافیه با `wget` دانلودشون کنیم.

```bash
for i in $(seq -w 1 10); do
    wget -P ~/Downloads "https://example.com/Better.Call.Saul.S05E$i.mkv"
done
```

بسته به سرور، امکان داره که خطای `403` بگیریم. برای حل این مشکل کافیه یک `user-agent` برای `wget` تنظیم کنیم.

```bash
for i in $(seq -w 1 10); do
    wget -P ~/Downloads --user-agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36' "https://example.com/Better.Call.Saul.S05E$i.mkv"
done
```

و تمام؛ همهٔ فایل‌ها به ترتیب دانلود می‌شن.