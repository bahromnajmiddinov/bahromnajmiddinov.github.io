# Web Server sozlash

## 3.1 Web Server nima uchun kerak?

1C:Enterprise HTTP servislari tashqi tizimlar uchun mavjud bo'lishi uchun web server orqali publikatsiya qilinadi.

### Tanlash:

| Web Server | Afzalliklari | Kamchiliklari | Tavsiya |
|------------|--------------|---------------|---------|
| **Apache HTTP Server** | ✅ Bepul<br>✅ Cross-platform<br>✅ Kuchli konfiguratsiya | ❌ Murakkab sozlash | Linux va test muhitlar uchun |
| **Windows IIS** | ✅ Windows bilan integratsiya<br>✅ Oson sozlash<br>✅ GUI mavjud | ❌ Faqat Windows<br>❌ Ba'zi limitlar | Windows production uchun |
| **nginx** | ✅ Yuqori performance<br>✅ Kam resurs iste'mol | ❌ 1C bilan to'g'ridan-to'g'ri integratsiya yo'q | Reverse proxy sifatida |

Ushbu qo'llanmada biz **Apache HTTP Server 2.4** bilan ishlashni ko'rib chiqamiz.

## 3.2 Apache HTTP Server yuklab olish

### Windows uchun

Apache'ning rasmiy saytida Windows uchun binary fayllar yo'q. Uchinchi tomon distributorlaridan olish kerak.

#### Variant 1: ApacheHaus (Tavsiya etiladi)

1. Brauzerda [https://www.apachehaus.com/cgi-bin/download.plx](https://www.apachehaus.com/cgi-bin/download.plx) ni oching

2. **Apache 2.4.xx Win64** versiyasini toping

3. Download tugmasini bosing (ZIP fayl yuklanadi)

#### Variant 2: Apache Lounge

1. [https://www.apachelounge.com/download/](https://www.apachelounge.com/download/) ga o'ting

2. **Apache 2.4.xx Win64** ni yuklab oling

#### Variant 3: Anindya (Installer bilan)

1. [https://www.anindya.com/apache-http-server-2-4-23-x86-32-bit-and-x64-64-bit-windows-installers/](https://www.anindya.com/apache-http-server-2-4-23-x86-32-bit-and-x64-64-bit-windows-installers/) ga boring

2. 64-bit installer'ni yuklab oling (`.msi` fayl)

3. Installer'ni ishga tushiring va wizard'ni kuzatib boring

**Eslatma:** Biz ApacheHaus ZIP variantidan foydalanamiz (ko'proq nazorat uchun).

### Qadam 1: ZIP faylni ochish

1. Yuklab olingan `httpd-2.4.xx-win64.zip` faylini extract qiling

2. Ichidan `Apache24` papkasini ko'chirib oling

3. Uni `C:\` drivega joylashtiring:
   ```
   C:\Apache24\
   ```

### Qadam 2: Tuzilmasi

```
C:\Apache24\
├── bin\              # Executable fayllar
│   ├── httpd.exe     # Asosiy server
│   ├── ab.exe        # Benchmarking tool
│   └── ...
├── conf\             # Konfiguratsiya fayllar
│   ├── httpd.conf    # Asosiy konfiguratsiya
│   ├── extra\        # Qo'shimcha konfiguratsiyalar
│   └── ...
├── htdocs\           # Web root (default sahifalar)
├── logs\             # Log fayllar
├── modules\          # Apache modullar
└── ...
```

## 3.3 Apache'ni sozlash

### Qadam 1: httpd.conf faylini ochish

1. `C:\Apache24\conf\httpd.conf` faylini matn muharririda oching

2. Yoki `notepad C:\Apache24\conf\httpd.conf` buyrug'ini ishga tushiring

### Qadam 2: Asosiy sozlamalar

**ServerRoot** o'rnatish (taxminan 37-qator):

```apache
# Before:
# ServerRoot "c:/Apache24"

# After:
ServerRoot "C:/Apache24"
```

**Listen port** tekshirish (taxminan 60-qator):

```apache
# Default:
Listen 80

# Agar port band bo'lsa, boshqa port ishlating:
# Listen 8080
```

**ServerName** o'rnatish (taxminan 227-qator):

```apache
# Before:
# ServerName www.example.com:80

# After:
ServerName localhost:80
```

### Qadam 3: Modullarni yoqish

1C bilan ishlash uchun kerakli modullar (taxminan 150-190-qatorlar):

```apache
# Kerakli modullar (# belgisini olib tashlang):
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule rewrite_module modules/mod_rewrite.so
LoadModule headers_module modules/mod_headers.so
```

### Qadam 4: DirectoryIndex

Default fayl nomlari (taxminan 283-qator):

```apache
<IfModule dir_module>
    DirectoryIndex index.html index.htm
</IfModule>
```

### Qadam 5: Saqlash

`httpd.conf` faylini saqlang (**Ctrl+S**).

## 3.4 1C bilan integratsiya

### Qadam 1: Virtual host yaratish

`httpd.conf` fayliga 1C uchun virtual host qo'shamiz.

Faylning oxiriga quyidagi konfiguratsiyani qo'shing:

```apache
#───────────────────────────────────────────────────────────
# 1C:Enterprise HTTP Services Configuration
#───────────────────────────────────────────────────────────

<VirtualHost *:80>
    ServerName localhost
    
    # 1C infobase nomi
    DocumentRoot "C:/Apache24/htdocs"
    
    # 1C HTTP service proxy
    ProxyPreserveHost On
    ProxyPass "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    ProxyPassReverse "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    
    # Logging
    ErrorLog "logs/1c-error.log"
    CustomLog "logs/1c-access.log" common
    
</VirtualHost>
```

**Tushuntirish:**

- `/radchenko/hs` - 1C HTTP servislar yo'li
  - `radchenko` - infobase nomi
  - `hs` - HTTP service prefiksi (hard-coded in 1C)
  
- `http://localhost:8080` - 1C web server porti
  - 1C default ravishda 8080 portda ishlaydi
  - Buni 1C configurator'da sozlash mumkin

### Qadam 2: ProxyPass va ProxyPassReverse

Bu direktiva Apache'ga aytadi:
- `/radchenko/hs` ga kelgan so'rovlarni `localhost:8080`ga yo'naltirsin
- 1C'dan qaytgan javoblarni teskari yo'naltirsin

```
Client                Apache               1C Server
  │                     │                     │
  ├──GET /radchenko/hs──►│                     │
  │                     ├─GET localhost:8080──►│
  │                     │                     │
  │                     │◄─────Response───────┤
  │◄────Response────────┤                     │
```

### Qadam 3: Bir nechta infobase

Agar bir nechta 1C infobase bo'lsa:

```apache
<VirtualHost *:80>
    ServerName localhost
    
    # Infobase 1: radchenko
    ProxyPass "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    ProxyPassReverse "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    
    # Infobase 2: production
    ProxyPass "/production/hs" "http://localhost:8081/production/hs"
    ProxyPassReverse "/production/hs" "http://localhost:8081/production/hs"
    
    # Infobase 3: test
    ProxyPass "/test/hs" "http://localhost:8082/test/hs"
    ProxyPassReverse "/test/hs" "http://localhost:8082/test/hs"
    
    ErrorLog "logs/1c-all-error.log"
    CustomLog "logs/1c-all-access.log" common
    
</VirtualHost>
```

## 3.5 Apache'ni Windows Service sifatida o'rnatish

### Qadam 1: Administrator Command Prompt

1. Start Menu'da **cmd** ni qidiring
2. **Command Prompt** ustiga o'ng tugma bosing
3. **Run as administrator** ni tanlang

### Qadam 2: Apache katalogiga o'tish

```cmd
cd C:\Apache24\bin
```

### Qadam 3: Service'ni o'rnatish

```cmd
httpd.exe -k install
```

Muvaffaqiyatli bo'lsa:
```
Installing the 'Apache2.4' service
The 'Apache2.4' service is successfully installed.
Testing httpd.conf....
Errors reported here must be corrected before the service can be started.
```

### Qadam 4: Service'ni ishga tushirish

```cmd
httpd.exe -k start
```

Yoki Windows Services orqali:

1. **Win+R** → `services.msc` → Enter
2. **Apache2.4** xizmatini toping
3. O'ng tugma → **Start**

### Qadam 5: Statusni tekshirish

Brauzerni oching va kiriting:
```
http://localhost
```

Apache default sahifasi ("It works!") ko'rinishi kerak.

## 3.6 1C Web Server sozlash

1C'da ham web server sozlash kerak.

### Qadam 1: 1C Server Agent ishga tushirish

1C training versiyasida server agent avtomatik ishga tushmaydi. Uni qo'lda ishga tushiramiz.

**Administrator Command Prompt** da:

```cmd
cd "C:\Program Files\1cv8\8.3.xx.xxxx\bin"
ragent.exe
```

Yoki Service sifatida o'rnatish:

```cmd
ragent.exe -srvc -regserver -port 1540 -d "C:\1C-Servers"
```

### Qadam 2: Infobase'ni Server rejimda qayta yaratish

Training versiyada server rejimi cheklangan, lekin HTTP servislar file rejimda ham ishlaydi.

### Qadam 3: 1C'da Web server sozlamalari

1C platformasida web server parametrlari odatda fayl rejimda avtomatik sozlanadi:

- **Port:** 8080 (default)
- **Root URL:** `/infobase_name/hs`

Bu sozlamalarni tekshirish uchun:

1. 1C Enterprise'ni ishchi rejimda oching
2. **Administration → Web Services** bo'limini oching (agar mavjud bo'lsa)

## 3.7 Test qilish

### Qadam 1: Oddiy test handler

1C module'da test handler yarating (agar hali yaratmagan bo'lsangiz):

```1c
Function TestGet(Request) Export
    Response = New HTTPServiceResponse(200);
    Response.Headers.Insert("Content-Type", "text/plain; charset=utf-8");
    Response.SetBodyFromString("Hello from 1C via Apache!");
    Return Response;
EndFunction
```

URL template: `/test` → Method: **GET** → Handler: `TestGet`

Configuration'ni yangilang (Ctrl+F7).

### Qadam 2: Brauzerdaн so'rov yuborish

Brauzerni oching:

```
http://localhost/radchenko/hs/exchange/test
```

**Kutilgan javob:**
```
Hello from 1C via Apache!
```

### Qadam 3: cURL orqali test

Command line'da:

```bash
curl http://localhost/radchenko/hs/exchange/test
```

Javob:
```
Hello from 1C via Apache!
```

### Qadam 4: POST so'rov test

```bash
curl -X POST http://localhost/radchenko/hs/exchange/test
```

## 3.8 Logging va monitoring

### Apache log fayllar

**Access log:**
```
C:\Apache24\logs\1c-access.log
```

Misol:
```
127.0.0.1 - - [15/Feb/2026:10:30:45 +0500] "GET /radchenko/hs/exchange/test HTTP/1.1" 200 28
```

**Error log:**
```
C:\Apache24\logs\1c-error.log
```

Misol:
```
[Mon Feb 15 10:30:45.123456 2026] [proxy:error] [pid 1234:tid 5678] [client 127.0.0.1:54321] AH00898: Error during SSL Handshake with remote server
```

### Loglarni real-time ko'rish

PowerShell'da:

```powershell
Get-Content C:\Apache24\logs\1c-access.log -Wait -Tail 20
```

Command Prompt'da:

```cmd
tail -f C:\Apache24\logs\1c-access.log
```

(agar Git Bash yoki WSL o'rnatilgan bo'lsa)

### 1C Technical Journal

1C'da batafsil loglar uchun Technical Journal'ni yoqing:

1. `C:\Users\YourName\AppData\Roaming\1C\1cv8\conf\logcfg.xml` faylini yarating:

```xml
<?xml version="1.0"?>
<config xmlns="http://v8.1c.ru/v8/tech-log">
    <log location="C:\1C-Logs" history="24">
        <event>
            <eq property="name" value="HTTP"/>
        </event>
        <property name="all"/>
    </log>
</config>
```

2. 1C platformasini qayta ishga tushiring

3. Loglar `C:\1C-Logs\` da paydo bo'ladi

## 3.9 Muammolarni hal qilish

### Muammo 1: "Apache service won't start"

**Tekshirish:**

```cmd
httpd.exe -t
```

Bu konfiguratsiya faylda xatoliklarni ko'rsatadi.

**Umumiy xatolar:**
- Sintaksis xatosi: qator va fayl ko'rsatiladi
- Port band: boshqa dastur 80 portda ishlayotgan bo'lishi mumkin
- Modul topilmadi: `LoadModule` yo'lini tekshiring

**Yechim:**
```cmd
# Portni tekshirish
netstat -ano | findstr :80

# Boshqa dasturni to'xtatish yoki Apache portini o'zgartirish
Listen 8080
```

### Muammo 2: "502 Bad Gateway"

**Sabab:** Apache 1C serverga ulana olmayapti.

**Tekshirish:**
1. 1C server agent ishlaydimi?
2. 1C infobase ochiq va ishlaydimi?
3. Port to'g'ri ko'rsatilganmi? (default: 8080)

**Yechim:**
```cmd
# 1C server agent'ni ishga tushiring
cd "C:\Program Files\1cv8\8.3.xx.xxxx\bin"
ragent.exe
```

### Muammo 3: "404 Not Found"

**Sabab:** URL noto'g'ri yoki HTTP servis publikatsiya qilinmagan.

**Tekshirish:**
1. URL to'g'ri: `/infobase_name/hs/service_name/template`
2. HTTP servis yaratilgan va configuration yangilangan (Ctrl+F7)
3. ProxyPass yo'li to'g'ri

**Yechim:**
- URL templateni tekshiring
- httpd.conf'da ProxyPass yo'lini tekshiring
- Apache'ni qayta yuklang: `httpd.exe -k restart`

### Muammo 4: "1C returns 500 error"

**Sabab:** 1C module'da kod xatosi.

**Yechim:**
1. 1C Designer'da moduleni oching
2. **Tools → Syntax Check** (Ctrl+Shift+F7)
3. Technical Journal loglarini tekshiring
4. Try-Except blok qo'shib xatolikni aniqlang

### Muammo 5: "Cyrillic characters broken"

**Sabab:** Encoding muammosi.

**Yechim:**

1C'da:
```1c
Response.Headers.Insert("Content-Type", "application/json; charset=utf-8");
```

Apache'da (`httpd.conf`):
```apache
AddDefaultCharset UTF-8
```

## 3.10 SSL/HTTPS sozlash (ixtiyoriy)

Production muhitda HTTPS tavsiya etiladi.

### Qadam 1: SSL modulini yoqish

`httpd.conf` da:

```apache
LoadModule ssl_module modules/mod_ssl.so
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
```

### Qadam 2: SSL konfiguratsiya faylini include qilish

```apache
Include conf/extra/httpd-ssl.conf
```

### Qadam 3: Self-signed sertifikat yaratish (test uchun)

OpenSSL bilan:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout C:/Apache24/conf/server.key \
  -out C:/Apache24/conf/server.crt
```

### Qadam 4: Virtual Host SSL

```apache
<VirtualHost *:443>
    ServerName localhost
    
    SSLEngine on
    SSLCertificateFile "C:/Apache24/conf/server.crt"
    SSLCertificateKeyFile "C:/Apache24/conf/server.key"
    
    ProxyPass "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    ProxyPassReverse "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    
</VirtualHost>
```

### Qadam 5: Apache'ni qayta yuklash

```cmd
httpd.exe -k restart
```

Test:
```
https://localhost/radchenko/hs/exchange/test
```

## 3.11 Performance tuning

### Apache sozlamalari

`httpd.conf` da:

```apache
# Timeout'ni kamaytirish
Timeout 60

# KeepAlive yoqish
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

# MPM prefork sozlamalari
<IfModule mpm_prefork_module>
    StartServers             5
    MinSpareServers          5
    MaxSpareServers         10
    MaxRequestWorkers      150
    MaxConnectionsPerChild   0
</IfModule>
```

### 1C sozlamalari

1C platformada:

- Connection pool o'lchamini oshiring
- Keshlashni optimallashtiring
- Indekslarni to'g'ri yarating

## 3.12 Windows IIS (qisqacha)

Windows Server'da IIS afzalroq bo'lishi mumkin.

### Asosiy qadamlar:

1. **IIS o'rnatish:**
   - Server Manager → Add Roles → Web Server (IIS)
   - Application Development → CGI va ISAPI Extensions

2. **URL Rewrite modulini o'rnatish:**
   - [https://www.iis.net/downloads/microsoft/url-rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)

3. **1C publication:**
   - 1C'da: Tools → Publish on Web Server → IIS
   - Virtual directory yaratiladi

4. **Test:**
   ```
   http://localhost/infobase_name/hs/service_name/template
   ```

## 3.13 Xulosa

Ushbu bo'limda siz:

✅ Apache HTTP Server yuklab olish va o'rnatishni o'rgandingiz  
✅ Apache'ni sozlashni va 1C bilan integratsiya qilishni o'rgandingiz  
✅ ProxyPass va reverse proxy'ni sozlashni o'rgandingiz  
✅ Logging va monitoring'ni sozlashni o'rgandingiz  
✅ Muammolarni hal qilish usullarini o'rgandingiz  

**Navbatdagi bo'lim:** [1C kod yozish](04-1c-code.md) - HTTP servislar uchun funksional kod yozish.

---

[← HTTP Servislari](02-http-services.md) | [Bosh sahifa](../README.md) | [1C Kod →](04-1c-code.md)