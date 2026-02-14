# 1C:Enterprise o'rnatish

## 1.1 Training versiyasi haqida

1C:Enterprise 8 platformasining bepul training versiyasi ishlab chiqish, test qilish va o'rganish uchun mo'ljallangan. Bu versiya to'liq funksional imkoniyatlarga ega, ammo ba'zi cheklovlar mavjud.

### Training versiyasining cheklovlari

- **Ma'lumotlar hajmi cheklangan:**
  - Hisob jadvallarida maksimal 2000 ta yozuv
  - Asosiy ob'ekt jadvallarida maksimal 2000 ta yozuv
  - Tabular bo'limlarda maksimal 1000 ta yozuv
  - Yozuvlar to'plamida maksimal 2000 ta yozuv
  - Tashqi ma'lumot manbalaridan maksimal 200 ta yozuv

- **Client/Server rejimi qo'llab-quvvatlanmaydi**
- **Taqsimlangan Infobase qo'llab-quvvatlanmaydi**
- **COM ulanish qo'llab-quvvatlanmaydi**
- **Parol va operatsion tizim autentifikatsiyasidan foydalanib bo'lmaydi**

⚠️ **Muhim:** Ishlab chiqarish muhitida (production) training versiyadan foydalanmang! Bu faqat o'rganish va test qilish uchun.

## 1.2 1C Training versiyasini yuklab olish

### Qadam 1: 1C Developer Network saytiga kirish

1. Brauzerda [https://1c-dn.com/developer_tools/1c_enterprise_8_platform_training_version/](https://1c-dn.com/developer_tools/1c_enterprise_8_platform_training_version/) manzilini oching

2. Agar hisobingiz bo'lmasa, **Register** tugmasini bosing va ro'yxatdan o'ting
   - Email manzilingizni kiriting
   - Parol yarating
   - Tasdiqlash emailini tekshiring

3. Agar hisobingiz bo'lsa, **Login** tugmasini bosing va tizimga kiring

### Qadam 2: Platformani yuklab olish

1. Sahifada **"Download 1C:Enterprise 8 platform (training version)"** havolasini toping

2. Operatsion tizimingizga mos versiyani tanlang:
   - **Windows** (32-bit yoki 64-bit)
   - **Linux** (DEB yoki RPM package)
   - **MacOS**

3. Yuklab olish tugmasini bosing

![1C Download Page](../images/01-download-page.png)

### Qadam 3: Faylni yuklab olish

- Fayl hajmi: taxminan 300-500 MB
- Yuklab olingan fayl: `setup.exe` (Windows uchun)
- Yuklab olish vaqti: Internet tezligiga bog'liq

## 1.3 O'rnatish jarayoni (Windows)

### Qadam 1: O'rnatish dasturini ishga tushirish

1. Yuklab olingan `setup.exe` faylini ikki marta bosing
2. Windows SmartScreen ogohlantirishi paydo bo'lishi mumkin:
   - **"More info"** ni bosing
   - **"Run anyway"** ni tanlang

### Qadam 2: O'rnatish wizard

1. **Welcome screen**
   - Tilni tanlang (Ingliz, Rus)
   - **Next** tugmasini bosing

2. **License Agreement**
   - Litsenziya shartlarini o'qing
   - **I accept the terms** belgisini qo'ying
   - **Next** tugmasini bosing

3. **Installation Type**
   - **Complete** (to'liq o'rnatish) - tavsiya etiladi
   - **Custom** (maxsus) - faqat kerakli komponentlar
   - **Next** tugmasini bosing

4. **Destination Folder**
   - Standart: `C:\Program Files\1cv8\`
   - O'zgartirish kerak bo'lsa **Browse** ni bosing
   - **Next** tugmasini bosing

5. **Ready to Install**
   - Sozlamalarni tekshiring
   - **Install** tugmasini bosing

6. **Kutish** (5-10 daqiqa)
   - O'rnatish jarayoni davom etadi
   - Progress bar ko'rsatiladi

7. **Completion**
   - **Finish** tugmasini bosing
   - Ish stolida 1C:Enterprise belgisi paydo bo'ladi

## 1.4 Birinchi ishga tushirish

### Qadam 1: 1C:Enterprise platformasini ochish

1. Ish stolida **1C:Enterprise** ikonkasini ikki marta bosing
2. Yoki: Start Menu → **1C:Enterprise 8 (training version)**

### Qadam 2: Infobase yaratish

1C platformasi ishga tushganda, **"1C:Enterprise startup"** oynasi ochiladi.

![1C Startup Window](../images/02-startup-window.png)

1. **Add...** tugmasini bosing

2. **"Add infobase to the list"** oynasida:
   - **Create new infobase** radio buttonni tanlang
   - **Next** tugmasini bosing

![Create Infobase Dialog](../images/03-create-infobase.png)

3. **Infobase type:**
   - **Standard configuration** - standart konfiguratsiya
   - **Demo base** - demo ma'lumotlar bilan
   - **Empty infobase** - bo'sh baza
   
   **Demo base**ni tanlash tavsiya etiladi (o'rganish uchun).

4. **Infobase settings:**
   - **Name:** `Infobase #1` (yoki istalgan nom)
   - **Location:** Fayl yo'lini tanlang (masalan: `C:\Users\YourName\Documents\InfoBase1`)
   - **Next** tugmasini bosing

5. **Finish** tugmasini bosing

### Qadam 3: Infobase ochish

1. Yaratilgan infobase ro'yxatda paydo bo'ladi
2. Uni belgilang
3. **1C:Enterprise** tugmasini bosing - ishchi rejimda ochish uchun
4. **Designer** tugmasini bosing - configurator rejimda ochish uchun

## 1.5 Demo Infobase import qilish (Radchenko)

Tayyor demo infobase bilan ishlash uchun:

### Qadam 1: GitHub repository'dan yuklab olish

```bash
# Git orqali
git clone https://github.com/sudomango/1C-Infobase-Radchenko

# Yoki ZIP fayl sifatida yuklab oling:
# https://github.com/sudomango/1C-Infobase-Radchenko/archive/refs/heads/master.zip
```

### Qadam 2: Infobase fayllarini joylashtirish

1. Yuklab olingan papkani oching
2. `1Cv8.1CD` faylini toping (bu asosiy database fayli)
3. Faylni qulay joyga ko'chiring (masalan: `C:\1C-Databases\Radchenko\`)

### Qadam 3: 1C'da mavjud infobase qo'shish

1. **1C:Enterprise startup** oynasini oching
2. **Add...** tugmasini bosing
3. **"Add infobase to the list"** oynasida:
   - **Add infobase** radio buttonni tanlang
   - **Next** tugmasini bosing

4. **Infobase location:**
   - **Browse** tugmasini bosing
   - `1Cv8.1CD` faylini tanlang
   - **Next** tugmasini bosing

5. **Infobase name:**
   - Nom kiriting: `Radchenko Demo`
   - **Finish** tugmasini bosing

### Qadam 4: Demo infobase bilan ishlash

1. Ro'yxatdan **Radchenko Demo** ni tanlang
2. **Designer** tugmasini bosing
3. Configurator oynasi ochiladi

![1C Designer](../images/04-designer-window.png)

Bu erda siz:
- Konfiguratsiya strukturasini ko'rishingiz
- HTTP servislarini yaratishingiz
- Kodlarni tahrirlashingiz
- Metadata bilan ishlashingiz mumkin

## 1.6 Muhim kataloglar va fayllar

### 1C o'rnatilgan joylar:

```
C:\Program Files\1cv8\
├── 8.3.25.1257\          # Versiya papkasi
│   ├── bin\              # Executable fayllar
│   │   ├── 1cv8.exe      # Asosiy dastur
│   │   ├── 1cv8c.exe     # Client
│   │   └── ragent.exe    # Server agent
│   ├── conf\             # Konfiguratsiya
│   └── licenses\         # Litsenziya fayllar
```

### Infobase fayllar:

```
C:\Users\YourName\Documents\InfoBase1\
├── 1Cv8.1CD              # Asosiy database fayl
├── 1Cv8.lgf              # Log file
├── 1Cv8.lgp              # Log settings
└── temp\                 # Vaqtinchalik fayllar
```

## 1.7 Muammolarni hal qilish

### Muammo: "Platform version not supported"

**Yechim:**
- Eng oxirgi training versiyasini yuklab oling
- Infobase faylini yangi platformaga moslashtiring

### Muammo: "File access denied"

**Yechim:**
- Administrator huquqlari bilan ishga tushiring
- Antivirus dasturini vaqtincha o'chiring
- Infobase fayliga to'liq ruxsat bering

### Muammo: "Cannot create infobase"

**Yechim:**
- Disk joyini tekshiring (kamida 1 GB bo'sh joy kerak)
- Fayl yo'lida maxsus belgilar yo'qligini tekshiring
- To'liq yo'l uzunligi 200 belgidan kam bo'lishi kerak

## 1.8 Keyingi qadamlar

✅ 1C:Enterprise o'rnatildi  
✅ Infobase yaratildi  
✅ Designer bilan tanishish  

**Navbatdagi bo'lim:** [HTTP Servislari yaratish](02-http-services.md)

---

[← Bosh sahifa](../README.md) | [HTTP Servislari →](02-http-services.md)