# دليل شامل لتثبيت وتشغيل Proxy في Termux لتسريع التحميل

## المقدمة

Proxy (البروكسي) هو خادم وسيط يساعد على:
- ✅ تسريع التحميل
- ✅ تجاوز الحظر الجغرافي
- ✅ توفير البيانات
- ✅ إخفاء عنوان IP

---

## القسم الأول: أنواع البروكسي المتاحة

### 1. HTTP Proxy
```bash
# بروكسي عادي للويب
# السرعة: متوسطة
# الاستهلاك: متوسط
```

### 2. SOCKS5 Proxy
```bash
# بروكسي متقدم وآمن
# السرعة: عالية
# الاستهلاك: منخفض
```

### 3. VPN
```bash
# شبكة خاصة افتراضية
# السرعة: متوسطة
# الأمان: عالي جداً
```

---

## القسم الثاني: تثبيت Termux بشكل صحيح أولاً

### الخطوة 1: تحديث المستودعات

```bash
# نظف الذاكرة المؤقتة أولاً
apt clean
rm -rf $PREFIX/var/lib/apt/lists/*

# عدل ملف المستودعات
nano $PREFIX/etc/apt/sources.list
```

**احذف كل شيء وأضف:**
```
deb https://packages.termux.dev/apt/termux-main stable main
```

**احفظ:** Ctrl + X, y, Enter

### الخطوة 2: تحديث النظام

```bash
pkg update
pkg upgrade -y
```

### الخطوة 3: تثبيت الأدوات الأساسية

```bash
pkg install -y git curl wget nano vim
```

---

## القسم الثالث: تثبيت Proxy Servers

### الحل 1: استخدام Tinyproxy (الأسهل)

#### التثبيت

```bash
# ثبت Tinyproxy
pkg install -y tinyproxy

# تحقق من التثبيت
tinyproxy --version
```

#### الإعدادات

```bash
# افتح ملف الإعدادات
nano $PREFIX/etc/tinyproxy/tinyproxy.conf
```

**أهم التغييرات:**

ابحث عن هذه الأسطر وعدّلها:

```bash
# تغيير المنفذ (غير 8888 إلى أي منفذ تريد)
Port 8888

# السماح بالاتصالات المحلية
# ابحث عن:
# Allow 127.0.0.1
# غيرها إلى:
Allow 0.0.0.0

# تفعيل الضغط
Compress Yes

# سرعة التخزين المؤقت
ConnectTimeout 600
```

**احفظ:** Ctrl + X, y, Enter

#### تشغيل البروكسي

```bash
# ابدأ البروكسي
tinyproxy -d

# تحقق من التشغيل
ps aux | grep tinyproxy

# اختبر الاتصال
curl -x 127.0.0.1:8888 http://google.com
```

---

### الحل 2: استخدام Squid (متقدم وأسرع)

#### التثبيت

```bash
# ثبت Squid
pkg install -y squid

# تحقق من التثبيت
squid --version
```

#### الإعدادات

```bash
# افتح ملف الإعدادات
nano $PREFIX/etc/squid/squid.conf
```

**إعدادات تسريع:**

ابحث عن هذه الأسطر وعدّلها:

```bash
# منفذ الاستماع
http_port 3128

# ذاكرة التخزين المؤقت
cache_mem 256 MB
maximum_object_size_in_memory 512 KB
maximum_object_size 200 MB

# سرعة الاتصال
half_closed_clients off
pconn_timeout 120 seconds

# تسريع DNS
dns_nameservers 8.8.8.8 8.8.4.4

# السماح بالاتصالات
acl localnet src 0.0.0.0/0
http_access allow localnet
http_access deny all
```

**احفظ:** Ctrl + X, y, Enter

#### تشغيل Squid

```bash
# أنشئ المجلدات المطلوبة
squid -z

# ابدأ Squid
squid -d 5

# تحقق من التشغيل
ps aux | grep squid

# اختبر الاتصال
curl -x 127.0.0.1:3128 http://google.com
```

---

### الحل 3: استخدام Privoxy (للوصول الآمن)

#### التثبيت

```bash
# ثبت Privoxy
pkg install -y privoxy

# تحقق
privoxy --version
```

#### الإعدادات

```bash
# افتح الإعدادات
nano $PREFIX/etc/privoxy/config
```

**تعديلات مهمة:**

```bash
# منفذ الاستماع
listen-address  127.0.0.1:8118

# السماح بالاتصالات البعيدة
listen-address  0.0.0.0:8118

# تفعيل الضغط
compression level 9
```

**احفظ:** Ctrl + X, y, Enter

#### تشغيل Privoxy

```bash
# ابدأ Privoxy
privoxy $PREFIX/etc/privoxy/config

# اختبر
curl -x 127.0.0.1:8118 http://google.com
```

---

## القسم الرابع: استخدام البروكسي مع Termux

### الطريقة 1: تعيين البروكسي عام في Termux

```bash
# اضبط متغيرات البيئة
export http_proxy=http://127.0.0.1:8888
export https_proxy=http://127.0.0.1:8888
export ftp_proxy=http://127.0.0.1:8888

# للتطبيقات التي تستخدم curl
export HTTP_PROXY=http://127.0.0.1:8888
export HTTPS_PROXY=http://127.0.0.1:8888
```

### الطريقة 2: استخدام البروكسي مع apt/pkg

```bash
# عدل ملف الإعدادات
nano ~/.aptrc
```

أضف:
```
Acquire::http::Proxy "http://127.0.0.1:8888/";
Acquire::https::Proxy "http://127.0.0.1:8888/";
```

**احفظ:** Ctrl + X, y, Enter

### الطريقة 3: استخدام البروكسي مع Git

```bash
# اضبط git
git config --global http.proxy http://127.0.0.1:8888
git config --global https.proxy http://127.0.0.1:8888

# تحقق
git config --global http.proxy
```

### الطريقة 4: استخدام البروكسي مع wget

```bash
# افتح الإعدادات
nano ~/.wgetrc
```

أضف:
```
http_proxy=http://127.0.0.1:8888/
https_proxy=http://127.0.0.1:8888/
ftp_proxy=http://127.0.0.1:8888/
```

---

## القسم الخامس: البروكسي المجاني عبر الإنترنت

### الطريقة 1: استخدام Free Proxy List

```bash
# ثبت أداة فحص البروكسي
pkg install -y proxychains-ng

# عدل الإعدادات
nano $PREFIX/etc/proxychains.conf
```

أضف بروكسي مجاني من:
- [Free Proxy List](https://www.freeproxylists.net/)
- [Proxy Sites](https://www.proxysites.com/)

**مثال:**
```
socks5 1.2.3.4 1080
http 5.6.7.8 8080
```

### الطريقة 2: استخدام Tor (البروكسي الآمن المجاني)

#### التثبيت

```bash
# ثبت Tor
pkg install -y tor

# تحقق
tor --version
```

#### التشغيل

```bash
# ابدأ Tor
tor

# افتح نافذة جديدة وتحقق
ps aux | grep tor
```

#### الاستخدام

```bash
# ثبت proxychains
pkg install -y proxychains-ng

# استخدم مع أي برنامج
proxychains curl http://google.com
```

---

## القسم السادس: برامج مجانية لتسريع WiFi

### 1. استخدام Speedtest

```bash
# ثبت Speedtest
pkg install -y speedtest-cli

# اختبر السرعة
speedtest-cli
```

### 2. مراقبة السرعة الحية

```bash
# ثبت أداة المراقبة
pkg install -y nethogs

# راقب الاستخدام
nethogs

# أو استخدم
iftop

# ثبت iftop
pkg install -y iftop
```

### 3. تحسين الأداء

```bash
# امسح الذاكرة
free -m
clear

# أغلق البرامج غير المستخدمة
ps aux

# قتل عملية محددة
kill -9 PID_number
```

---

## القسم السابع: دليل البدء السريع

### تشغيل سريع - الخطوة بخطوة

#### الخطوة 1: فتح Termux

```bash
# ابدأ بتحديث النظام
pkg update && pkg upgrade -y
```

#### الخطوة 2: ثبت البروكسي المفضل

```bash
# الخيار الأول - Tinyproxy (الأسهل)
pkg install -y tinyproxy

# أو الخيار الثاني - Squid (الأسرع)
pkg install -y squid
```

#### الخطوة 3: شغل البروكسي

**لـ Tinyproxy:**
```bash
tinyproxy -d
```

**لـ Squid:**
```bash
squid -z
squid -d 5
```

#### الخطوة 4: اختبر البروكسي

```bash
# افتح نافذة جديدة واكتب:
curl -x 127.0.0.1:8888 http://google.com

# يجب أن ترى محتوى الصفحة
```

#### الخطوة 5: استخدم البروكسي

```bash
# لـ pkg
export http_proxy=http://127.0.0.1:8888
pkg install -y git

# تحميل أسرع!
```

---

## القسم الثامن: أوامر مهمة

### التحكم بالبروكسي

```bash
# عرض العمليات الجارية
ps aux | grep proxy
ps aux | grep tinyproxy
ps aux | grep squid

# إيقاف البروكسي
pkill tinyproxy
pkill squid

# إعادة تشغيل البروكسي
pkill tinyproxy && tinyproxy -d

# عرض سجلات البروكسي
tail -f $PREFIX/var/log/tinyproxy/tinyproxy.log
```

### اختبار السرعة

```bash
# اختبر بدون بروكسي
curl -o /dev/null -s -w "%{time_total}\n" http://google.com

# اختبر مع بروكسي
curl -x 127.0.0.1:8888 -o /dev/null -s -w "%{time_total}\n" http://google.com

# قارن الوقت!
```

---

## القسم التاسع: استكشاف الأخطاء

### المشكلة 1: "Connection refused"

```bash
# تأكد من تشغيل البروكسي
ps aux | grep proxy

# إعادة التشغيل
pkill tinyproxy
tinyproxy -d

# اختبر من جديد
curl -x 127.0.0.1:8888 http://google.com
```

### المشكلة 2: البروكسي بطيء

```bash
# قلل عدد الاتصالات
# غير في ملف الإعدادات:
# ConnectTimeout 300 (بدلاً من 600)
# MaxConnections 50 (بدلاً من 100)

# أعد التشغيل
pkill tinyproxy
tinyproxy -d
```

### المشكلة 3: استهلاك بيانات عالي

```bash
# فعّل الضغط في الإعدادات
nano $PREFIX/etc/tinyproxy/tinyproxy.conf

# أضف/عدّل:
Compress Yes
```

### المشكلة 4: منفذ مشغول

```bash
# ابحث عن ما يستخدم المنفذ
lsof -i :8888

# اقتل العملية
kill -9 PID_number

# جرب منفذ مختلف
# عدّل Port 9999 في الإعدادات
```

---

## القسم العاشر: نصائح تسريع متقدمة

### 1. ضبط DNS

```bash
# عدّل إعدادات DNS
nano $PREFIX/etc/resolv.conf

# أضف:
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
nameserver 1.0.0.1
```

### 2. تفعيل IPv6 (إذا كان متاحاً)

```bash
# في ملف إعدادات البروكسي
Allow 0.0.0.0/0
Allow ::/0
```

### 3. تحسين سرعة WiFi

```bash
# قرّب جهازك من الراوتر
# غيّر قناة WiFi في إعدادات الراوتر
# استخدم 2.4 GHz بدلاً من 5 GHz في المناطق الضيقة
# استخدم 5 GHz في المناطق المفتوحة
```

### 4. تقليل استهلاك البيانات

```bash
# ثبت أداة ضغط الصور
pkg install -y jpegoptim
pkg install -y optipng

# استخدم proxychains مع curl المضغوط
curl --compressed http://google.com
```

---

## القسم الحادي عشر: أتمتة البروكسي (خيارات متقدمة)

### إنشاء سكريبت لتشغيل البروكسي تلقائياً

```bash
# أنشئ ملف سكريبت
nano ~/start_proxy.sh
```

أضف:
```bash
#!/bin/bash

# تحقق من تشغيل البروكسي
if pgrep -x "tinyproxy" > /dev/null
then
    echo "البروكسي يعمل بالفعل"
else
    echo "تشغيل البروكسي..."
    tinyproxy -d
    echo "تم تشغيل البروكسي على المنفذ 8888"
fi

# اضبط متغيرات البيئة
export http_proxy=http://127.0.0.1:8888
export https_proxy=http://127.0.0.1:8888

echo "تم ضبط متغيرات البروكسي"
```

**احفظ:** Ctrl + X, y, Enter

### جعل السكريبت قابلاً للتنفيذ

```bash
chmod +x ~/start_proxy.sh

# شغله
./start_proxy.sh
```

---

## القسم الثاني عشر: مقارنة سريعة

| الميزة | Tinyproxy | Squid | Privoxy | Tor |
|--------|-----------|-------|---------|-----|
| السهولة | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| السرعة | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| الأمان | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| استهلاك الموارد | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| المجاني | ✅ | ✅ | ✅ | ✅ |

---

## القسم الثالث عشر: أوامر نهائية مهمة

### التشغيل الكامل من البداية

```bash
# 1. تحديث النظام
pkg update && pkg upgrade -y

# 2. تثبيت Git
pkg install -y git

# 3. تثبيت البروكسي
pkg install -y tinyproxy

# 4. تشغيل البروكسي
tinyproxy -d

# 5. اختبر التحميل
wget -O /dev/null http://google.com
```

### التحقق من الأداء

```bash
# قبل البروكسي
time wget -O /dev/null http://example.com

# بعد البروكسي
export http_proxy=http://127.0.0.1:8888
time wget -O /dev/null http://example.com

# قارن الوقت!
```

---

## الخلاصة

✅ **ابدأ بـ Tinyproxy** - الأسهل والأسرع للمبتدئين
✅ **استخدم Squid** - إذا أردت أداء عالي
✅ **استخدم Tor** - إذا أردت أمان وخصوصية
✅ **فعّل الضغط** - لتقليل استهلاك البيانات
✅ **اختبر دائماً** - للتأكد من التحسن

---

**مستعد! جرب الآن وأخبرني بالنتيجة! 🚀**
