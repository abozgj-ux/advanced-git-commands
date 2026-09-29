# تثبيت Kali Linux على Termux X11 - الدليل الشامل

## المشكلة الحالية
```
Failed to install: Unauthorized: 'kali' does not exist or is a private image
```

**السبب:** proot-distro لا يدعم Kali مباشرة الآن.

---

## ✅ الحل 1: استخدام Ubuntu + أدوات Kali (الأسهل)

### الخطوة 1: تثبيت Ubuntu

```bash
proot-distro install ubuntu:24.04
```

الانتظار حتى ينتهي (5-10 دقائق).

### الخطوة 2: دخول Ubuntu

```bash
proot-distro login ubuntu:24.04 --shared-tmp
```

### الخطوة 3: تحديث Ubuntu

داخل Ubuntu:

```bash
apt update
apt upgrade -y
```

### الخطوة 4: إضافة مستودع Kali

```bash
# أضف مستودع Kali
wget -q https://archive.kali.org/archive-key.asc -O - | apt-key add -

echo "deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware" >> /etc/apt/sources.list.d/kali.list

# تحديث
apt update
```

### الخطوة 5: تثبيت أدوات Kali

```bash
# تثبيت أدوات شهيرة
apt install -y kali-tools-wireless kali-tools-web nmap metasploit-framework aircrack-ng

# أو كل الأدوات (يأخذ مساحة!)
# apt install -y kali-linux
```

### الخطوة 6: تثبيت XFCE Desktop

```bash
apt install -y xfce4 xfce4-terminal dbus xterm
```

---

## ✅ الحل 2: تثبيت Kali من صورة يدوياً (متقدم)

إذا كنت تريد Kali الحقيقي، جرب هذا:

### تحميل صورة Kali من GitHub

```bash
cd ~/storage/downloads

# تحميل صورة Kali المضغوطة
wget https://github.com/termux/proot-distro/releases/download/v5.9.0/kali-arm64.tar.xz

# فك الضغط
tar -xf kali-arm64.tar.xz

# نقل للمكان الصحيح
mkdir -p $PREFIX/var/lib/proot-distro/installed-rootfs/kali
mv * $PREFIX/var/lib/proot-distro/installed-rootfs/kali/
```

### التحقق

```bash
proot-distro list
# يجب أن ترى kali
```

### دخول Kali

```bash
proot-distro login kali --shared-tmp
```

---

## ✅ الحل 3: استخدام Debian + Kali (الأفضل)

### الخطوة 1: تثبيت Debian

```bash
proot-distro install debian:bookworm
```

### الخطوة 2: دخول Debian

```bash
proot-distro login debian:bookworm --shared-tmp
```

### الخطوة 3: تحديث

```bash
apt update
apt upgrade -y
```

### الخطوة 4: إضافة مستودع Kali

```bash
# أضف مستودع Kali
apt-key adv --keyserver hkp://keys.openpgp.org --recv-keys ED444FF07D8D0BF6

echo "deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware" > /etc/apt/sources.list.d/kali.list

apt update
```

### الخطوة 5: تثبيت أدوات Kali المهمة

```bash
# الأدوات الأساسية
apt install -y nmap metasploit-framework wireshark aircrack-ng burpsuite

# أو أدوات محددة
apt install -y sqlmap nikto hashcat john-the-ripper
```

### الخطوة 6: تثبيت سطح المكتب

```bash
apt install -y xfce4 xfce4-terminal dbus xterm
```

---

## 🚀 تشغيل مع X11

### من Termux (نافذة 1):

```bash
termux-x11 :1 -ac &
```

افتح تطبيق **Termux:X11**.

### من Termux (نافذة 2):

```bash
proot-distro login ubuntu:24.04 --shared-tmp
```

### داخل Ubuntu/Debian:

```bash
export DISPLAY=:1
export PULSE_SERVER=127.0.0.1
startxfce4 &
```

---

## 📋 أوامر مفيدة

### داخل التوزيعة:

```bash
# عرض الأدوات المتثبتة
dpkg -l | grep kali

# تحديث أدوات Kali
apt update
apt install -y kali-tools-web

# تشغيل أداة معينة
nmap -h
msfconsole
wireshark &
```

### من Termux:

```bash
# عرض التوزيعات المثبتة
proot-distro list

# دخول التوزيعة
proot-distro login ubuntu:24.04

# حذف التوزيعة
proot-distro remove ubuntu:24.04

# تشغيل أمر مباشر
proot-distro login ubuntu:24.04 -- apt update
```

---

## جدول المقارنة

| الحل | السهولة | السرعة | توفر الأدوات | المساحة |
|-----|--------|--------|-------------|--------|
| Ubuntu + Kali Tools | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 2-3 GB |
| Debian + Kali Tools | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 1.5-2 GB |
| Kali اليدوي | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 3-5 GB |

---

## 🎯 التوصية النهائية

**استخدم الحل 1 (Ubuntu + Kali Tools)** لأنه:
- ✅ الأسهل والأسرع
- ✅ يحتوي على جميع أدوات Kali المهمة
- ✅ يعمل بسلاسة مع X11
- ✅ استهلاك موارد منخفض

---

## سكريبت سريع للتثبيت الكامل

أنشئ ملف:

```bash
nano ~/install-kali-tools.sh
```

ضع بداخله:

```bash
#!/bin/bash

echo "📦 تثبيت Ubuntu..."
proot-distro install ubuntu:24.04

echo "📲 دخول Ubuntu..."
proot-distro login ubuntu:24.04 --shared-tmp -- bash << 'EOF'
apt update
apt upgrade -y

echo "🔧 إضافة مستودع Kali..."
wget -q https://archive.kali.org/archive-key.asc -O - | apt-key add -
echo "deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware" >> /etc/apt/sources.list.d/kali.list

apt update

echo "⚙️ تثبيت أدوات Kali..."
apt install -y kali-tools-web nmap metasploit-framework aircrack-ng

echo "🖥️ تثبيت XFCE Desktop..."
apt install -y xfce4 xfce4-terminal dbus xterm

echo "✅ تم التثبيت بنجاح!"
EOF
```

احفظ: `Ctrl + X`, `y`, `Enter`

شغّله:

```bash
chmod +x ~/install-kali-tools.sh
bash ~/install-kali-tools.sh
```

---

جرب الآن! أي حل تفضل؟ 🚀
