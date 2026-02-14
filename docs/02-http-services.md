# HTTP Servislari yaratish

## 2.1 HTTP Servis nima?

HTTP servis - bu 1C:Enterprise platformasida yaratilgan web service bo'lib, tashqi tizimlar (Odoo, Python, JavaScript va h.k.) bilan HTTP protokoli orqali ma'lumot almashish imkonini beradi.

### HTTP servisning asosiy qismlari:

```
HTTP Service
├── URL Template (masalan: /exchange)
├── HTTP Methods
│   ├── GET    (ma'lumot olish)
│   ├── POST   (ma'lumot yuborish)
│   ├── PUT    (yangilash)
│   └── DELETE (o'chirish)
└── Handler Module (1C kod)
```

### Ishlash sxemasi:

```
┌──────────────┐                           ┌──────────────┐
│   Odoo/      │   HTTP Request            │     1C       │
│   Client     ├──────────────────────────►│   HTTP       │
│              │   GET/POST /exchange      │   Service    │
│              │                           │              │
│              │   HTTP Response           │              │
│              │◄──────────────────────────┤   Handler    │
│              │   JSON/XML data           │   Module     │
└──────────────┘                           └──────────────┘
```

## 2.2 HTTP Servis yaratish (Designer orqali)

### Qadam 1: Designer'ni ochish

1. **1C:Enterprise startup** oynasida infobase'ni tanlang
2. **Designer** tugmasini bosing
3. Configurator oynasi ochiladi

![Designer Window](../images/05-designer-interface.png)

### Qadam 2: HTTP Servislar bo'limini topish

1. Chap panelda **Configuration** daraxtini oching
2. Quyidagi tartibda:
   - **Common** ▼
   - **Web services** ▼
   - **HTTP services** ▼

![Configuration Tree](../images/06-http-services-tree.png)

### Qadam 3: Yangi HTTP servis yaratish

1. **HTTP services** ustiga o'ng tugma bosing
2. **New** ni tanlang yoki **Insert** tugmasini bosing

![Create HTTP Service](../images/07-http-service-create.png)

### Qadam 4: HTTP servisni sozlash

**"HTTP service exchange"** oynasi ochiladi. Asosiy sozlamalar:

#### Main tab (Asosiy):

- **Name:** `exchange` (ichki nom)
- **Synonym:** `Exchange` (ko'rinadigan nom)
- **Comment:** `HTTP service for Odoo integration` (izoh)
- **Root URL:** `exchange` (URL yo'li)

```
To'liq URL: http://localhost/your_base/hs/exchange
                                         └─────┘
                                         Root URL
```

#### Subsystems tab:

Bu bo'limda servisni qaysi subsystem'ga tegishli ekanligini belgilaysiz (ixtiyoriy).

#### Sessions tab:

Session sozlamalari (odatda standart qoldiriladi).

### Qadam 5: Saqlash

1. **OK** tugmasini bosing yoki **Ctrl+S** bosing
2. HTTP servis yaratildi va configuratsiya daraxtida ko'rinadi

## 2.3 URL Templates yaratish

URL Template - bu HTTP so'rovlarini qabul qiladigan endpoint (nuqta).

### Qadam 1: URL Templates bo'limini ochish

1. Yaratilgan `exchange` HTTP servisini oching (ikki marta bosing)
2. Chap panelda **URL templates** bo'limiga o'ting

![URL Templates Panel](../images/08-url-templates.png)

### Qadam 2: Yangi template yaratish

1. **URL templates** qatorida **+** (plus) belgisini bosing yoki o'ng tugma → **New**

2. URL template sozlamalari:

**Template:** `/counterparty` (yoki istalgan nom)

Bu quyidagi URLni yaratadi:
```
http://localhost/your_base/hs/exchange/counterparty
```

3. **OK** tugmasini bosing

### Qadam 3: Bir nechta template yaratish

Turli resurslar uchun bir nechta template yaratishingiz mumkin:

| Template | Maqsad | To'liq URL |
|----------|--------|------------|
| `/counterparty` | Kontragentlar | `.../hs/exchange/counterparty` |
| `/product` | Mahsulotlar | `.../hs/exchange/product` |
| `/invoice` | Hisob-fakturalar | `.../hs/exchange/invoice` |
| `/stock` | Ombor | `.../hs/exchange/stock` |
| `/salesorder` | Savdo buyurtmalari | `.../hs/exchange/salesorder` |

![Multiple Templates](../images/09-multiple-templates.png)

## 2.4 HTTP Methods qo'shish

Har bir URL template uchun HTTP metodlarini (GET, POST, PUT, DELETE) sozlash kerak.

### Qadam 1: Method qo'shish

1. **URL templates** ro'yxatida `/counterparty` ni tanlang
2. Yuqori panelda **URL templates** oynasi ochiladi
3. O'ng panelda **HTTP method** qismini toping

![HTTP Methods](../images/10-http-methods.png)

### Qadam 2: POST method qo'shish

1. **HTTP method** dropdown'dan **POST** ni tanlang
2. **Handler:** `CounterpartyPOST` (handler nomi) ni kiriting

Bu handler module'da yaratilgan funksiya nomi bo'ladi.

### Qadam 3: GET method qo'shish

Xuddi shunday tartibda:

1. **+** belgisini bosing (agar bir nechta method kerak bo'lsa)
2. **HTTP method:** **GET**
3. **Handler:** `CounterpartyGET`

### Qadam 4: Barcha methodlar

Har bir URL template uchun kerakli methodlarni qo'shing:

#### `/counterparty`:
- **GET** → `CounterpartyGET` - Kontragent ma'lumotlarini olish
- **POST** → `CounterpartyPOST` - Yangi kontragent yaratish

#### `/product`:
- **GET** → `ProductGET` - Mahsulot ma'lumotlarini olish
- **POST** → `ProductPOST` - Yangi mahsulot yaratish

#### `/salesorder`:
- **POST** → `SalesOrderPOST` - Savdo buyurtmasini yaratish

## 2.5 Handler Module yaratish

HTTP so'rovlarni qayta ishlash uchun modul kodi yoziladi.

### Qadam 1: Module ochish

1. HTTP servis oynasida **URL templates** bo'limini tanlang
2. Yuqorida **Subsystems**, **URL templates**, **Sessions**, **Other** tugmalari bor
3. **Other** tugmasini bosing

![Other Tab](../images/11-module-tab.png)

### Qadam 2: Module oynasi

**Module** qismi ochiladi. Bu yerda 1C kod yozasiz.

![Module Editor](../images/12-module-editor.png)

### Qadam 3: Asosiy struktura

Module faylida barcha handlerlar yoziladi:

```1c
// INTERFACE SECTION - SINGLE FILE
// NO INTEGRATION - SINGLE FILE

//═══════════════════════════════════════════════════════════
//# CounterpartyGET(Zapros) Eksport
//═══════════════════════════════════════════════════════════

Function CounterpartyGET(Request)
    
    Response = New HTTPServiceResponse(200);
    
    // TODO: Ma'lumotlarni olish va qaytarish
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# CounterpartyPOST(Zapros) Eksport
//═══════════════════════════════════════════════════════════

Function CounterpartyPOST(Request)
    
    Response = New HTTPServiceResponse(200);
    
    // TODO: Yangi kontragent yaratish
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# ProductPOST(Zapros) Eksport
//═══════════════════════════════════════════════════════════

Function ProductPOST(Request)
    
    Response = New HTTPServiceResponse(200);
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# ProductGET(Zapros) Eksport  
//═══════════════════════════════════════════════════════════

Function ProductGET(Request)
    
    Response = New HTTPServiceResponse(200);
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# SalesOrderPOST(Zapros) Eksport
//═══════════════════════════════════════════════════════════

Function SalesOrderPOST(Request)
    
    Response = New HTTPServiceResponse(200);
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# InvoicePOST(Zapros) Eksport
//═══════════════════════════════════════════════════════════

Function InvoicePOST(Request)
    
    Response = New HTTPServiceResponse(200);
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# StockGET(Zapros) Eksport
//═══════════════════════════════════════════════════════════

Function StockGET(Request)
    
    Response = New HTTPServiceResponse(200);
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# TestGet(Zapros) Eksport
//═══════════════════════════════════════════════════════════

Function TestGet(Request)
    
    Response = New HTTPServiceResponse(200);
    Response.SetBodyFromString("Test GET method is working!");
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# TestPost(Zapros) Eksport
//═══════════════════════════════════════════════════════════

Function TestPost(Request)
    
    Response = New HTTPServiceResponse(200);
    Response.SetBodyFromString("Test POST method is working!");
    
    Return Response;
    
EndFunction
```

### Qadam 4: Saqlash

1. **Ctrl+S** bosing yoki **File → Save** tanlang
2. Module saqlandi

## 2.6 HTTP Servisni publikatsiya qilish

HTTP servisdan foydalanish uchun uni publikatsiya qilish kerak.

### Qadam 1: Configuration menusini ochish

1. Yuqori menyudan **Configuration** ni tanlang
2. **Update Database Configuration** ni bosing

Yoki klaviaturada: **Ctrl+F7**

![Update Configuration](../images/13-update-config.png)

### Qadam 2: Kutish

Database yangilanishi 10-30 soniya davom etadi. Progress bar ko'rsatiladi.

### Qadam 3: Tasdiqlash

"Database configuration updated successfully" xabari paydo bo'ladi.

**OK** tugmasini bosing.

## 2.7 HTTP Servisni tekshirish

### Qadam 1: Test endpoint yaratish

Avval moduleda oddiy test handler yarating:

```1c
Function TestGet(Request)
    Response = New HTTPServiceResponse(200);
    Response.SetBodyFromString("Hello from 1C!");
    Return Response;
EndFunction
```

URL template qo'shing: `/test` → Method: **GET** → Handler: `TestGet`

### Qadam 2: Web serverdan test qilish

Agar web server sozlangan bo'lsa (keyingi bo'limda):

```bash
curl http://localhost/your_base/hs/exchange/test
```

Javob:
```
Hello from 1C!
```

### Qadam 3: Brauzerdaн test

Brauzerni oching:
```
http://localhost/your_base/hs/exchange/test
```

Sahifada "Hello from 1C!" ko'rinishi kerak.

## 2.8 HTTP Response kodlari

1C'da to'g'ri HTTP response kodlarini qaytarish muhim:

| Kod | Ma'nosi | Qachon ishlatiladi |
|-----|---------|-------------------|
| **200** | OK | So'rov muvaffaqiyatli bajarildi |
| **201** | Created | Yangi resurs yaratildi |
| **400** | Bad Request | Noto'g'ri so'rov (validatsiya xatosi) |
| **401** | Unauthorized | Autentifikatsiya kerak |
| **403** | Forbidden | Ruxsat yo'q |
| **404** | Not Found | Resurs topilmadi |
| **500** | Internal Server Error | Server xatosi |

Misol:

```1c
Function ProductGET(Request)
    
    Try
        // Ma'lumotni olish
        ProductData = GetProductData();
        
        Response = New HTTPServiceResponse(200);
        Response.SetBodyFromString(ProductData);
        
    Except
        // Xatolik yuz berdi
        Response = New HTTPServiceResponse(500);
        Response.SetBodyFromString("Internal error: " + ErrorDescription());
    EndTry;
    
    Return Response;
    
EndFunction
```

## 2.9 HTTP Headers

Response headerlari o'rnatish:

```1c
Function CounterpartyGET(Request)
    
    Response = New HTTPServiceResponse(200);
    
    // Content-Type headerini o'rnatish
    Response.Headers.Insert("Content-Type", "application/json; charset=utf-8");
    
    // CORS headerlari (agar kerak bo'lsa)
    Response.Headers.Insert("Access-Control-Allow-Origin", "*");
    Response.Headers.Insert("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
    
    // Ma'lumotni qaytarish
    JSONData = "{""name"": ""Test Counterparty""}";
    Response.SetBodyFromString(JSONData);
    
    Return Response;
    
EndFunction
```

## 2.10 Request parametrlarini olish

### URL Query parametrlari

```
GET /hs/exchange/counterparty?id=12345&name=Test
```

1C kodida:

```1c
Function CounterpartyGET(Request)
    
    // Query parametrlarini olish
    ID = Request.URLParameters.Get("id");
    Name = Request.URLParameters.Get("name");
    
    // Tekshirish
    If ID = Undefined Then
        Response = New HTTPServiceResponse(400);
        Response.SetBodyFromString("Parameter 'id' is required");
        Return Response;
    EndIf;
    
    // Davom etish...
    Response = New HTTPServiceResponse(200);
    Return Response;
    
EndFunction
```

### POST Body'dan ma'lumot olish

```1c
Function CounterpartyPOST(Request)
    
    // Request body'ni o'qish
    RequestBody = Request.GetBodyAsString();
    
    // JSON parse qilish
    JSONReader = New JSONReader();
    JSONReader.SetString(RequestBody);
    Data = ReadJSON(JSONReader);
    JSONReader.Close();
    
    // Ma'lumotlarni olish
    CounterpartyName = Data.Get("name");
    CounterpartyINN = Data.Get("inn");
    
    // TODO: Kontragent yaratish
    
    Response = New HTTPServiceResponse(201);
    Return Response;
    
EndFunction
```

## 2.11 Muhim eslatmalar

⚠️ **Eslatmalar:**

1. **Eksport keyword:** Barcha handler funksiyalar `Eksport` (Export) keyword bilan e'lon qilinishi kerak:
   ```1c
   Function TestGet(Request) Export
   ```

2. **Nom mos kelishi:** URL template'dagi handler nomi module'dagi funksiya nomi bilan aynan mos kelishi kerak.

3. **Request parametri:** Har bir handler `Request` parametrini qabul qiladi.

4. **Response qaytarish:** Har bir handler `HTTPServiceResponse` ob'ektini qaytarishi shart.

5. **Encoding:** UTF-8 kodlashdan foydalaning (ayniqsa kirill harflar uchun).

## 2.12 Xatolarni hal qilish

### Muammo: "Handler not found"

**Yechim:**
- Handler nomi to'g'ri yozilganligini tekshiring
- `Export` keywordni qo'shganingizni tekshiring
- Configuration'ni yangilashni unutmang (Ctrl+F7)

### Muammo: "HTTP service not accessible"

**Yechim:**
- Web server to'g'ri sozlanganligini tekshiring
- URL yo'lini to'g'ri kirityapsizmi tekshiring
- Firewall va antivirus sozlamalarini tekshiring

### Muammo: "500 Internal Server Error"

**Yechim:**
- Module kodidagi sintaksis xatolarini tekshiring
- Try-Except blok qo'shib xatolikni aniqlang
- 1C Technical Journal'da loglarni ko'ring

## 2.13 Xulosa

Ushbu bo'limda siz:

✅ HTTP servis yaratishni o'rgandingiz  
✅ URL templates sozlashni o'rgandingiz  
✅ HTTP methods qo'shishni o'rgandingiz  
✅ Handler module yozishni o'rgandingiz  
✅ Servisni publikatsiya qilishni o'rgandingiz  

**Navbatdagi bo'lim:** [Web Server sozlash](03-web-server-setup.md) - Apache yoki IIS orqali HTTP servislarni ochish.

---

[← O'rnatish](01-installation.md) | [Bosh sahifa](../README.md) | [Web Server →](03-web-server-setup.md)