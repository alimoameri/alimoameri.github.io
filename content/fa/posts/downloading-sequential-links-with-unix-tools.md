+++
date = '2025-05-11T20:16:41+03:30'
draft = false
title = 'دانلود فایل‌های ترتیبی با ابزارهای unix'
summary= "در این پست به این می‌پردازیم که چطور با ابزارهای سادهٔ unix لینک‌های دانلود فایل‌های ترتیبی مثال بازی یا سریال رو با هم دانلود کنیم."
tags= ["unix", "wget", "tr", "seq", "linux"]
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

می‌خواستم خیلی سریع و ساده همهٔ فایل‌ها رو دانلود کنم. راه حل اصولیش اینه که بشینیم یک اسکریپت بنویسیم که url بگیره، صفحه رو scrape، لینک‌های دانلود رو استخراج و دانلود کنه. ولی از اونجایی که حوصله این کار رو ندارم با ابزارهای لینوکسی سریع کارم رو راه انداختم. تنها کاری که باید بکنیم اینه که تمامی رو لینک‌ها به ترتیب شمارشون تولید کنیم. بعدش می‌شه با ابزاری مثل `wget` به راحتی دانلودشون کرد.

## ایدهٔ اولیه: استفاده از unix brace expantion

اولین چیزی که به ذهنم رسید استفاده از *unix brace expantion* برای تولید لینک‌ها بود ( از اونجایی که `echo` از کاراکتر فاصله برای جدا کردن خروجی استفاده می‌کنه، یک پردازش نهایی با `tr` هم لازمه):

```bash
> echo https://example.com/Better.Call.Saul.S05E{1..10}.mkv | tr ' ' "\n"
https://example.com/Better.Call.Saul.S05E1.mkv
https://example.com/Better.Call.Saul.S05E2.mkv
https://example.com/Better.Call.Saul.S05E3.mkv
https://example.com/Better.Call.Saul.S05E4.mkv
https://example.com/Better.Call.Saul.S05E5.mkv
https://example.com/Better.Call.Saul.S05E6.mkv
https://example.com/Better.Call.Saul.S05E7.mkv
https://example.com/Better.Call.Saul.S05E8.mkv
https://example.com/Better.Call.Saul.S05E9.mkv
https://example.com/Better.Call.Saul.S05E10.mkv
```

اگر این روش به درستی لینک‌ها رو تولید کنه، کافیه به جای `echo` دستور `wget` بزاریم و تمام. ولی اگر به عدد مشخص شده برای شمارهٔ لینک‌های اصلی دقت کنیم می‌بینیم که اعداد zero pad هستن:

```bash
https://example.com/Better.Call.Saul.S05E01.mkv
https://example.com/Better.Call.Saul.S05E02.mkv
https://example.com/Better.Call.Saul.S05E03.mkv
...
```

پس باید بریم سراغ یک روش دیگه.

## `seq`

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