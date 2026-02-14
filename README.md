# 1C:Enterprise va Odoo Integratsiyasi

1C:Enterprise platformasi va Odoo ERP tizimi o'rtasida HTTP servislari orqali integratsiya qilish bo'yicha to'liq qo'llanma.

## Mundarija

1. [Kirish](#kirish)
2. [1C:Enterprise o'rnatish](docs/01-installation.md)
3. [HTTP Servislari yaratish](docs/02-http-services.md)
4. [Web Server sozlash](docs/03-web-server-setup.md)
5. [1C kod yozish](docs/04-1c-code.md)
6. [Odoo bilan integratsiya](docs/05-odoo-integration.md)
7. [Autentifikatsiya](docs/06-authentication.md)
8. [Misollar va Amaliy qo'llanmalar](docs/07-examples.md)

## Kirish

Ushbu qo'llanma 1C:Enterprise 8 platformasi va Odoo ERP tizimi o'rtasida RESTful HTTP servislari orqali integratsiya yaratish jarayonini bosqichma-bosqich tushuntiradi.

### Texnologiyalar

- **1C:Enterprise 8** (Training Version) - Backend biznes logika
- **Apache HTTP Server** yoki **Windows IIS** - Web server
- **Odoo** - ERP tizimi
- **HTTP/HTTPS** - Ma'lumot almashinuvi protokoli
- **JSON/XML** - Ma'lumot formatlari

### Integratsiya arxitekturasi

```
┌─────────────┐         HTTP/HTTPS          ┌─────────────┐
│             │◄──────────────────────────►│             │
│    Odoo     │    GET/POST Requests       │ 1C:Enterprise│
│   (Client)  │    JSON/XML Response       │   (Server)  │
│             │                             │             │
└─────────────┘                             └─────────────┘
       │                                           │
       │                                           │
       ▼                                           ▼
  Python Code                              1C HTTP Service
  xmlrpc/jsonrpc                           Module + Handler
```

## Asosiy xususiyatlar

✅ **Bepul Training versiyadan foydalanish**  
✅ **HTTP servislari yaratish va sozlash**  
✅ **GET va POST metodlari**  
✅ **JSON/XML ma'lumot formatlari**  
✅ **Xavfsiz autentifikatsiya**  
✅ **Real-time ma'lumot almashinuvi**  

## Tezkor Boshlash

### 1. 1C Training versiyasini yuklab oling

[1C Developer Tools sahifasiga](https://1c-dn.com/developer_tools/1c_enterprise_8_platform_training_version/) o'ting va royxatdan o'ting.

### 2. Apache Server o'rnating

Apache HTTP Server 2.4 versiyasini [rasmiy saytdan](https://httpd.apache.org/download.cgi) yuklab oling.

### 3. Demo Infobase import qiling

```bash
git clone https://github.com/sudomango/1C-Infobase-Radchenko
```

### 4. HTTP servislarini sozlang

1C Configurator orqali HTTP servislarini yarating va publikatsiya qiling.

### 5. Odoo bilan ulang

Odoo'dan Python orqali HTTP so'rovlarini yuboring.

## Qo'llab-quvvatlash

Savollar yoki muammolar bo'lsa, [Issues](https://github.com/yourusername/yourrepo/issues) bo'limida murojaat qiling.

## Litsenziya

Ushbu qo'llanma MIT litsenziyasi ostida tarqatiladi.

## Mualliflar

- Sizning ismingiz
- Hissa qo'shganlar

---

**Keyingi qadam:** [1C:Enterprise o'rnatish](docs/01-installation.md)
