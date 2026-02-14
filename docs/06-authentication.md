# Autentifikatsiya

## 6.1 Umumiy ma'lumot

HTTP servislarga kirish huquqini cheklash va xavfsizlikni ta'minlash uchun autentifikatsiya zarur.

### Autentifikatsiya turlari:

| Turi | Xavfsizlik darajasi | Qiyinlik | Tavsiya |
|------|-------------------|----------|---------|
| **Basic Auth** | O'rta | Oson | Test muhit |
| **API Key** | O'rta | O'rtacha | Production (HTTP) |
| **OAuth 2.0** | Yuqori | Murakkab | Enterprise |
| **JWT Token** | Yuqori | O'rtacha | Modern API |

Ushbu qo'llanmada biz **Basic Auth** va **API Key** yondashuvlarini ko'rib chiqamiz.

## 6.2 Basic Authentication

### Asosiy tushuncha

Basic Auth - bu HTTP protokolining standart autentifikatsiya mexanizmi.

```
Authorization: Basic base64(username:password)
```

### 6.2.1 Apache'da Basic Auth

#### Qadam 1: htpasswd yaratish

```bash
cd C:\Apache24\bin

# Foydalanuvchi yaratish
htpasswd.exe -c C:\Apache24\conf\.htpasswd admin

# Parol kiritish
New password: ********
Re-type new password: ********
```

#### Qadam 2: Qo'shimcha foydalanuvchilar

```bash
# -c ni ishlatmang (fayl ustiga yoziladi)
htpasswd.exe C:\Apache24\conf\.htpasswd user2
```

#### Qadam 3: Apache konfiguratsiyasi

`httpd.conf` faylida:

```apache
<VirtualHost *:80>
    ServerName localhost
    
    # 1C HTTP service
    <Location "/radchenko/hs">
        AuthType Basic
        AuthName "1C Integration API"
        AuthUserFile "C:/Apache24/conf/.htpasswd"
        Require valid-user
    </Location>
    
    ProxyPass "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    ProxyPassReverse "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    
</VirtualHost>
```

#### Qadam 4: Apache'ni qayta yuklash

```cmd
httpd.exe -k restart
```

#### Qadam 5: Test qilish

**Autentifikatsiyasiz (401 xatolik):**

```bash
curl http://localhost/radchenko/hs/exchange/test
```

Javob:
```html
401 Unauthorized
```

**Autentifikatsiya bilan:**

```bash
curl -u admin:password http://localhost/radchenko/hs/exchange/test
```

Javob:
```
Hello from 1C!
```

### 6.2.2 Python (requests) da Basic Auth

```python
import requests

url = "http://localhost/radchenko/hs/exchange/counterparty"

# Usul 1: auth parametr
response = requests.get(url, auth=('admin', 'password'))

# Usul 2: headers orqali
import base64

credentials = base64.b64encode(b'admin:password').decode('utf-8')
headers = {
    'Authorization': f'Basic {credentials}'
}
response = requests.get(url, headers=headers)

print(response.status_code)
print(response.text)
```

### 6.2.3 Odoo'da Basic Auth

```python
from odoo import models
import requests


class Integration1C(models.Model):
    _name = 'integration.1c'
    
    username = fields.Char('Username', required=True)
    password = fields.Char('Password', required=True)
    
    def _get_auth(self):
        """Return authentication tuple"""
        return (self.username, self.password)
    
    def get_counterparties(self):
        """Get counterparties with authentication"""
        url = f"{self.base_url}/counterparty"
        
        response = requests.get(
            url,
            auth=self._get_auth(),
            timeout=30
        )
        
        response.raise_for_status()
        return response.json()
```

## 6.3 API Key Authentication

API Key - bu har bir clientga berilgan maxsus kalit.

### 6.3.1 1C'da API Key tekshirish

#### HTTP Service Module'da:

```1c
//═══════════════════════════════════════════════════════════
//# ValidateAPIKey - API key tekshirish
//═══════════════════════════════════════════════════════════

Function ValidateAPIKey(Request)
    
    // ─── Header'dan API key olish ───
    APIKey = Request.Headers.Get("X-API-Key");
    
    If IsBlankString(APIKey) Then
        Return False;
    EndIf;
    
    // ─── Database'dan tekshirish ───
    Query = New Query;
    Query.Text = 
    "SELECT
    |    APIKeys.Ref AS Ref,
    |    APIKeys.Active AS Active,
    |    APIKeys.ExpiryDate AS ExpiryDate
    |FROM
    |    Catalog.APIKeys AS APIKeys
    |WHERE
    |    APIKeys.Key = &Key";
    
    Query.SetParameter("Key", APIKey);
    
    Result = Query.Execute();
    Selection = Result.Select();
    
    If Not Selection.Next() Then
        Return False;  // Key topilmadi
    EndIf;
    
    If Not Selection.Active Then
        Return False;  // Key faol emas
    EndIf;
    
    If Selection.ExpiryDate <> Date(1,1,1) And Selection.ExpiryDate < CurrentDate() Then
        Return False;  // Key muddati o'tgan
    EndIf;
    
    Return True;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# UnauthorizedResponse - 401 javob
//═══════════════════════════════════════════════════════════

Function UnauthorizedResponse()
    
    ErrorData = New Structure;
    ErrorData.Insert("error", True);
    ErrorData.Insert("message", "Invalid or missing API key");
    ErrorData.Insert("code", 401);
    
    JSONWriter = New JSONWriter;
    JSONWriter.SetString();
    WriteJSON(JSONWriter, ErrorData);
    JSONString = JSONWriter.Close();
    
    Response = New HTTPServiceResponse(401);
    Response.Headers.Insert("Content-Type", "application/json; charset=utf-8");
    Response.Headers.Insert("WWW-Authenticate", "API-Key");
    Response.SetBodyFromString(JSONString);
    
    Return Response;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# CounterpartyGET - API key bilan himoyalangan
//═══════════════════════════════════════════════════════════

Function CounterpartyGET(Request) Export
    
    // ─── API key tekshirish ───
    If Not ValidateAPIKey(Request) Then
        Return UnauthorizedResponse();
    EndIf;
    
    // ─── Asosiy logika ───
    Try
        CounterpartyList = GetCounterpartyList();
        
        ResultData = New Structure;
        ResultData.Insert("items", CounterpartyList);
        ResultData.Insert("count", CounterpartyList.Count());
        
        Return SuccessResponse(200, ResultData);
        
    Except
        Return ErrorResponse(500, ErrorDescription());
    EndTry;
    
EndFunction
```

### 6.3.2 API Keys catalog yaratish

1C'da yangi catalog yarating: **Catalog.APIKeys**

**Maydonlar:**

| Maydon | Tur | Tavsif |
|--------|-----|--------|
| **Description** | String(100) | Nom/Tavsif |
| **Key** | String(255) | API kalit (UUID) |
| **Active** | Boolean | Faol/Nofaol |
| **ExpiryDate** | Date | Amal qilish muddati |
| **Client** | String(100) | Client nomi |
| **CreatedDate** | Date | Yaratilgan sana |

**API Key generatsiya:**

```1c
Function GenerateAPIKey()
    Return String(New UUID());
EndFunction
```

### 6.3.3 Python'da API Key

```python
import requests

url = "http://localhost/radchenko/hs/exchange/counterparty"

# API key
api_key = "550e8400-e29b-41d4-a716-446655440000"

headers = {
    'X-API-Key': api_key,
    'Content-Type': 'application/json'
}

response = requests.get(url, headers=headers)

if response.status_code == 401:
    print("Invalid API key")
elif response.status_code == 200:
    data = response.json()
    print(f"Success: {data}")
else:
    print(f"Error: {response.status_code}")
```

### 6.3.4 Odoo'da API Key

```python
from odoo import models, fields


class Integration1C(models.Model):
    _name = 'integration.1c'
    
    api_key = fields.Char('API Key', required=True)
    
    def _get_headers(self):
        """Return HTTP headers with API key"""
        return {
            'Content-Type': 'application/json; charset=utf-8',
            'X-API-Key': self.api_key
        }
    
    def _make_request(self, method, endpoint, data=None, params=None):
        """Make HTTP request with API key"""
        url = f"{self.base_url}{endpoint}"
        
        try:
            response = requests.request(
                method=method,
                url=url,
                json=data,
                params=params,
                headers=self._get_headers(),
                timeout=30
            )
            
            if response.status_code == 401:
                raise UserError('Invalid API Key')
            
            response.raise_for_status()
            return response
            
        except requests.exceptions.RequestException as e:
            _logger.error(f"1C Request Error: {e}")
            raise
```

## 6.4 JWT Token (Advanced)

JSON Web Token - modern va xavfsiz autentifikatsiya usuli.

### 6.4.1 JWT strukturasi

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiMTIzIiwiZXhwIjoxNjQ1MDB9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
│                                      │                                  │
│          Header (base64)             │      Payload (base64)            │  Signature
```

### 6.4.2 1C'da JWT yaratish (sodda)

**Eslatma:** 1C'da to'liq JWT qo'llab-quvvatlash yo'q. Sodda variant:

```1c
Function CreateSimpleToken(UserID)
    
    // ─── Token strukturasi ───
    TokenData = New Structure;
    TokenData.Insert("user_id", UserID);
    TokenData.Insert("issued_at", CurrentUniversalDate());
    TokenData.Insert("expires_at", CurrentUniversalDate() + 3600);  // 1 soat
    
    // ─── JSON ───
    JSONWriter = New JSONWriter;
    JSONWriter.SetString();
    WriteJSON(JSONWriter, TokenData);
    JSONString = JSONWriter.Close();
    
    // ─── Base64 encode ───
    BinaryData = GetBinaryDataFromString(JSONString, "UTF-8");
    TokenBase64 = Base64String(BinaryData);
    
    // ─── Signature (oddiy hash) ───
    SecretKey = "MySecretKey123";
    SignatureData = TokenBase64 + SecretKey;
    Hash = ComputeHash(SignatureData, HashFunction.MD5);
    
    // ─── To'liq token ───
    Token = TokenBase64 + "." + Hash;
    
    Return Token;
    
EndFunction
```

### 6.4.3 Token tekshirish

```1c
Function ValidateToken(Token)
    
    Try
        // ─── Token ajratish ───
        Parts = StrSplit(Token, ".", False);
        If Parts.Count() <> 2 Then
            Return False;
        EndIf;
        
        TokenBase64 = Parts[0];
        ReceivedHash = Parts[1];
        
        // ─── Signature tekshirish ───
        SecretKey = "MySecretKey123";
        SignatureData = TokenBase64 + SecretKey;
        ExpectedHash = ComputeHash(SignatureData, HashFunction.MD5);
        
        If ReceivedHash <> ExpectedHash Then
            Return False;  // Signature mos kelmadi
        EndIf;
        
        // ─── Token decode ───
        BinaryData = Base64Value(TokenBase64);
        JSONString = GetStringFromBinaryData(BinaryData, "UTF-8");
        
        JSONReader = New JSONReader;
        JSONReader.SetString(JSONString);
        TokenData = ReadJSON(JSONReader);
        JSONReader.Close();
        
        // ─── Expiry tekshirish ───
        ExpiresAt = TokenData.Get("expires_at");
        If ExpiresAt < CurrentUniversalDate() Then
            Return False;  // Token muddati o'tgan
        EndIf;
        
        Return True;
        
    Except
        Return False;
    EndTry;
    
EndFunction
```

### 6.4.4 Python'da JWT (PyJWT kutubxonasi)

```bash
pip install pyjwt
```

```python
import jwt
import datetime

# Token yaratish
secret_key = "MySecretKey123"

payload = {
    'user_id': '12345',
    'exp': datetime.datetime.utcnow() + datetime.timedelta(hours=1)
}

token = jwt.encode(payload, secret_key, algorithm='HS256')
print(f"Token: {token}")

# Token tekshirish
try:
    decoded = jwt.decode(token, secret_key, algorithms=['HS256'])
    print(f"Valid token: {decoded}")
except jwt.ExpiredSignatureError:
    print("Token expired")
except jwt.InvalidTokenError:
    print("Invalid token")
```

## 6.5 HTTPS (SSL/TLS)

Production muhitda HTTPS zarur.

### 6.5.1 Apache SSL konfiguratsiyasi

```apache
<VirtualHost *:443>
    ServerName api.example.com
    
    # SSL Engine
    SSLEngine on
    SSLCertificateFile "/path/to/certificate.crt"
    SSLCertificateKeyFile "/path/to/private.key"
    SSLCertificateChainFile "/path/to/ca-bundle.crt"
    
    # SSL Protocol
    SSLProtocol all -SSLv2 -SSLv3
    SSLCipherSuite HIGH:!aNULL:!MD5
    
    # 1C Proxy
    ProxyPass "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    ProxyPassReverse "/radchenko/hs" "http://localhost:8080/radchenko/hs"
    
    # Logs
    ErrorLog "logs/ssl-error.log"
    CustomLog "logs/ssl-access.log" combined
    
</VirtualHost>
```

### 6.5.2 Let's Encrypt sertifikat (bepul)

**Linux'da:**

```bash
# Certbot o'rnatish
sudo apt-get install certbot python3-certbot-apache

# Sertifikat olish
sudo certbot --apache -d api.example.com

# Avtomatik yangilanish
sudo certbot renew --dry-run
```

**Windows'da:**

1. [Win-ACME](https://www.win-acme.com/) yuklab oling
2. Dasturni ishga tushiring
3. Domain nomini kiriting
4. Apache variant tanlang

## 6.6 IP Whitelist

Faqat ma'lum IP manzillardan kirish huquqi.

### 6.6.1 Apache'da IP cheklash

```apache
<Location "/radchenko/hs">
    # Faqat bu IP'lardan ruxsat
    Require ip 192.168.1.0/24
    Require ip 10.0.0.50
    Require ip 203.0.113.5
    
    # Boshqa barcha IP'lar rad etiladi
</Location>
```

### 6.6.2 1C'da IP tekshirish

```1c
Function ValidateClientIP(Request)
    
    // ─── Client IP olish ───
    ClientIP = Request.Headers.Get("X-Forwarded-For");
    If IsBlankString(ClientIP) Then
        ClientIP = Request.Headers.Get("Remote-Addr");
    EndIf;
    
    // ─── Ruxsat berilgan IP'lar ro'yxati ───
    AllowedIPs = New Array;
    AllowedIPs.Add("192.168.1.100");
    AllowedIPs.Add("10.0.0.50");
    AllowedIPs.Add("203.0.113.5");
    
    // ─── Tekshirish ───
    If AllowedIPs.Find(ClientIP) = Undefined Then
        Return False;
    EndIf;
    
    Return True;
    
EndFunction
```

## 6.7 Rate Limiting

Bir clientdan juda ko'p so'rov kelishini oldini olish.

### 6.7.1 Apache mod_ratelimit

```apache
<Location "/radchenko/hs">
    # 100 KB/s limit
    SetOutputFilter RATE_LIMIT
    SetEnv rate-limit 100
</Location>
```

### 6.7.2 1C'da oddiy rate limit

```1c
// ─── Global o'zgaruvchi ───
Var RequestCounter;

Function CheckRateLimit(ClientID)
    
    If RequestCounter = Undefined Then
        RequestCounter = New Map;
    EndIf;
    
    // ─── Client uchun counter ───
    CurrentTime = CurrentUniversalDate();
    ClientData = RequestCounter.Get(ClientID);
    
    If ClientData = Undefined Then
        // Yangi client
        ClientData = New Structure;
        ClientData.Insert("count", 1);
        ClientData.Insert("window_start", CurrentTime);
        RequestCounter.Insert(ClientID, ClientData);
        Return True;
    EndIf;
    
    // ─── Time window (1 daqiqa) ───
    WindowDuration = 60;  // seconds
    If (CurrentTime - ClientData.window_start) > WindowDuration Then
        // Yangi window
        ClientData.count = 1;
        ClientData.window_start = CurrentTime;
        Return True;
    EndIf;
    
    // ─── Limit tekshirish (60 so'rov/daqiqa) ───
    MaxRequests = 60;
    If ClientData.count >= MaxRequests Then
        Return False;  // Limit oshdi
    EndIf;
    
    ClientData.count = ClientData.count + 1;
    Return True;
    
EndFunction
```

## 6.8 Best Practices

### ✅ Xavfsizlik tavsiyalari:

1. **HTTPS ishlatish** - Har doim production'da
2. **Kuchli parollar** - Kamida 12 belgi, murakkab
3. **API Key'larni himoya qilish** - Environment variables'da saqlash
4. **Token expiry** - Muddatli tokenlar ishlatish
5. **Rate limiting** - Abuse'dan himoya
6. **IP whitelist** - Agar mumkin bo'lsa
7. **Loglarni monitoring qilish** - Suspicious activity aniqlash
8. **CORS sozlash** - Faqat kerakli domainlar

### ❌ Xato yondashuvlar:

1. HTTP orqali sensitive ma'lumot yuborish
2. Parollarni kodda saqlash
3. Token'larni expire qilmaslik
4. Rate limit qo'ymaslik
5. Barcha IP'lardan ruxsat berish
6. Loglarni yozmaslik

## 6.9 Xulosa

Ushbu bo'limda siz:

✅ Basic Authentication'ni sozlashni o'rgandingiz  
✅ API Key autentifikatsiyani amalga oshirishni o'rgandingiz  
✅ JWT token konseptsiyasini tushundingiz  
✅ HTTPS va SSL'ni sozlashni o'rgandingiz  
✅ IP whitelist va rate limiting'ni qo'shishni o'rgandingiz  

**Navbatdagi bo'lim:** [Misollar va Amaliy qo'llanmalar](07-examples.md)

---

[← Odoo Integration](05-odoo-integration.md) | [Bosh sahifa](../README.md) | [Examples →](07-examples.md)