# 1C Kod yozish

## 4.1 HTTP Service Module strukturasi

HTTP service module 1C kod muhiti bo'lib, tashqi tizimlardan kelgan so'rovlarni qayta ishlaydi.

### Asosiy komponentlar:

```1c
// ═══════════════════════════════════════════════════════════
// HTTP SERVICE MODULE: exchange
// ═══════════════════════════════════════════════════════════

// ┌─────────────────────────────────────────────────────────┐
// │ 1. HANDLER FUNCTIONS (Export)                           │
// └─────────────────────────────────────────────────────────┘

// ┌─────────────────────────────────────────────────────────┐
// │ 2. HELPER FUNCTIONS (Internal)                          │
// └─────────────────────────────────────────────────────────┘

// ┌─────────────────────────────────────────────────────────┐
// │ 3. DATA PROCESSING FUNCTIONS                            │
// └─────────────────────────────────────────────────────────┘

// ┌─────────────────────────────────────────────────────────┐
// │ 4. VALIDATION FUNCTIONS                                 │
// └─────────────────────────────────────────────────────────┘
```

## 4.2 Request va Response ob'ektlari

### Request ob'ekti

Tashqi tizimdan kelgan so'rov ma'lumotlarini o'z ichiga oladi:

```1c
Function ExampleHandler(Request) Export
    
    // ─── URL parametrlari ───
    ID = Request.URLParameters.Get("id");           // ?id=123
    Filter = Request.URLParameters.Get("filter");   // &filter=active
    
    // ─── HTTP Headers ───
    ContentType = Request.Headers.Get("Content-Type");
    AuthToken = Request.Headers.Get("Authorization");
    
    // ─── Request Body ───
    Body = Request.GetBodyAsString();              // JSON yoki XML
    BodyBytes = Request.GetBodyAsBinaryData();     // Binary ma'lumot
    
    // ─── HTTP Method ───
    Method = Request.HTTPMethod;                   // "GET", "POST", etc.
    
    // ─── Base URL ───
    BaseURL = Request.BaseURL;                     // "/radchenko/hs/exchange"
    
    // ─── Relative URL ───
    RelativeURL = Request.RelativeURL;             // "/counterparty?id=123"
    
    Response = New HTTPServiceResponse(200);
    Return Response;
    
EndFunction
```

### Response ob'ekti

1C'dan qaytariladigan javob:

```1c
Function ExampleHandler(Request) Export
    
    // ─── Response yaratish ───
    Response = New HTTPServiceResponse(200);  // Status code
    
    // ─── Headers o'rnatish ───
    Response.Headers.Insert("Content-Type", "application/json; charset=utf-8");
    Response.Headers.Insert("X-Custom-Header", "MyValue");
    
    // ─── Body o'rnatish (text) ───
    JSONString = "{""status"": ""success""}";
    Response.SetBodyFromString(JSONString);
    
    // ─── Body o'rnatish (binary) ───
    // Response.SetBodyFromBinaryData(BinaryData);
    
    // ─── Status code o'rnatish ───
    Response.StatusCode = 200;  // Yoki constructorda
    
    // ─── Reason phrase (ixtiyoriy) ───
    Response.Reason = "OK";
    
    Return Response;
    
EndFunction
```

## 4.3 JSON bilan ishlash

### JSON yaratish (ObjectToJSON)

1C ob'ektini JSON formatga o'girish:

```1c
Function CounterpartyGET(Request) Export
    
    // ─── Ma'lumotlar strukturasi ───
    Data = New Structure;
    Data.Insert("id", 12345);
    Data.Insert("name", "Test Counterparty");
    Data.Insert("inn", "1234567890");
    Data.Insert("active", True);
    
    // ─── JSON'ga o'girish ───
    JSONWriter = New JSONWriter;
    JSONWriter.SetString();
    WriteJSON(JSONWriter, Data);
    JSONString = JSONWriter.Close();
    
    // ─── Response qaytarish ───
    Response = New HTTPServiceResponse(200);
    Response.Headers.Insert("Content-Type", "application/json; charset=utf-8");
    Response.SetBodyFromString(JSONString);
    
    Return Response;
    
EndFunction
```

**Natija:**
```json
{
  "id": 12345,
  "name": "Test Counterparty",
  "inn": "1234567890",
  "active": true
}
```

### JSON parse qilish (JSONToObject)

Tashqi tizimdan kelgan JSON'ni o'qish:

```1c
Function CounterpartyPOST(Request) Export
    
    // ─── Request body'ni olish ───
    RequestBody = Request.GetBodyAsString();
    
    // ─── JSON'ni parse qilish ───
    JSONReader = New JSONReader;
    JSONReader.SetString(RequestBody);
    Data = ReadJSON(JSONReader);
    JSONReader.Close();
    
    // ─── Ma'lumotlarni olish ───
    Name = Data.Get("name");
    INN = Data.Get("inn");
    Address = Data.Get("address");
    
    // ─── Validatsiya ───
    If IsBlankString(Name) Then
        Return ErrorResponse(400, "Name is required");
    EndIf;
    
    // ─── Kontragent yaratish ───
    NewCounterparty = CreateCounterparty(Name, INN, Address);
    
    // ─── Muvaffaqiyatli javob ───
    ResultData = New Structure;
    ResultData.Insert("id", NewCounterparty.Ref);
    ResultData.Insert("status", "created");
    
    Return SuccessResponse(201, ResultData);
    
EndFunction
```

### Ichki funksiya: SuccessResponse

```1c
Function SuccessResponse(StatusCode, Data)
    
    JSONWriter = New JSONWriter;
    JSONWriter.SetString();
    WriteJSON(JSONWriter, Data);
    JSONString = JSONWriter.Close();
    
    Response = New HTTPServiceResponse(StatusCode);
    Response.Headers.Insert("Content-Type", "application/json; charset=utf-8");
    Response.SetBodyFromString(JSONString);
    
    Return Response;
    
EndFunction
```

### Ichki funksiya: ErrorResponse

```1c
Function ErrorResponse(StatusCode, ErrorMessage)
    
    ErrorData = New Structure;
    ErrorData.Insert("error", True);
    ErrorData.Insert("message", ErrorMessage);
    ErrorData.Insert("code", StatusCode);
    
    JSONWriter = New JSONWriter;
    JSONWriter.SetString();
    WriteJSON(JSONWriter, ErrorData);
    JSONString = JSONWriter.Close();
    
    Response = New HTTPServiceResponse(StatusCode);
    Response.Headers.Insert("Content-Type", "application/json; charset=utf-8");
    Response.SetBodyFromString(JSONString);
    
    Return Response;
    
EndFunction
```

## 4.4 Ma'lumotlar bazasi bilan ishlash

### Kontragent yaratish (Catalog)

```1c
Function CreateCounterparty(Name, INN, Address)
    
    // ─── Yangi ob'ekt yaratish ───
    CounterpartyObj = Catalogs.Counterparties.CreateItem();
    
    // ─── Maydonlarni to'ldirish ───
    CounterpartyObj.Description = Name;
    CounterpartyObj.INN = INN;
    CounterpartyObj.Address = Address;
    CounterpartyObj.Date = CurrentDate();
    
    // ─── Saqlash ───
    Try
        CounterpartyObj.Write();
    Except
        Raise "Counterparty save failed: " + ErrorDescription();
    EndTry;
    
    Return CounterpartyObj;
    
EndFunction
```

### Kontragent yangilash

```1c
Function UpdateCounterparty(CounterpartyRef, NewData)
    
    // ─── Ob'ektni olish ───
    CounterpartyObj = CounterpartyRef.GetObject();
    
    If CounterpartyObj = Undefined Then
        Raise "Counterparty not found";
    EndIf;
    
    // ─── Maydonlarni yangilash ───
    If NewData.Property("name") Then
        CounterpartyObj.Description = NewData.name;
    EndIf;
    
    If NewData.Property("inn") Then
        CounterpartyObj.INN = NewData.inn;
    EndIf;
    
    If NewData.Property("address") Then
        CounterpartyObj.Address = NewData.address;
    EndIf;
    
    // ─── Saqlash ───
    CounterpartyObj.Write();
    
    Return CounterpartyObj;
    
EndFunction
```

### Ma'lumotlarni o'qish (Query)

```1c
Function GetCounterpartyList(FilterActive = True)
    
    // ─── Query yaratish ───
    Query = New Query;
    Query.Text = 
    "SELECT
    |    Counterparties.Ref AS Ref,
    |    Counterparties.Description AS Name,
    |    Counterparties.INN AS INN,
    |    Counterparties.Address AS Address,
    |    Counterparties.IsActive AS Active
    |FROM
    |    Catalog.Counterparties AS Counterparties
    |WHERE
    |    Counterparties.IsActive = &Active";
    
    Query.SetParameter("Active", FilterActive);
    
    // ─── Query'ni bajarish ───
    Result = Query.Execute();
    Selection = Result.Select();
    
    // ─── Massivga to'plash ───
    CounterpartyArray = New Array;
    
    While Selection.Next() Do
        
        Item = New Structure;
        Item.Insert("id", String(Selection.Ref.UUID()));
        Item.Insert("name", Selection.Name);
        Item.Insert("inn", Selection.INN);
        Item.Insert("address", Selection.Address);
        Item.Insert("active", Selection.Active);
        
        CounterpartyArray.Add(Item);
        
    EndDo;
    
    Return CounterpartyArray;
    
EndFunction
```

### Handler: Ro'yxatni qaytarish

```1c
Function CounterpartyGET(Request) Export
    
    // ─── URL parametridan filter olish ───
    ActiveFilter = Request.URLParameters.Get("active");
    
    FilterValue = True;  // Default
    If ActiveFilter = "false" Or ActiveFilter = "0" Then
        FilterValue = False;
    EndIf;
    
    // ─── Ma'lumotlarni olish ───
    CounterpartyList = GetCounterpartyList(FilterValue);
    
    // ─── JSON formatda qaytarish ───
    ResultData = New Structure;
    ResultData.Insert("count", CounterpartyList.Count());
    ResultData.Insert("items", CounterpartyList);
    
    Return SuccessResponse(200, ResultData);
    
EndFunction
```

## 4.5 Document yaratish (Savdo buyurtmasi)

```1c
Function CreateSalesOrder(CustomerRef, ProductsArray)
    
    // ─── Yangi hujjat yaratish ───
    SalesOrderObj = Documents.SalesOrders.CreateDocument();
    
    // ─── Sarlavha maydonlari ───
    SalesOrderObj.Date = CurrentDate();
    SalesOrderObj.Customer = CustomerRef;
    SalesOrderObj.Number = GetNextDocumentNumber("SalesOrders");
    
    // ─── Tabular qism (mahsulotlar) ───
    For Each ProductItem In ProductsArray Do
        
        NewRow = SalesOrderObj.Products.Add();
        NewRow.Product = GetProductByCode(ProductItem.code);
        NewRow.Quantity = ProductItem.quantity;
        NewRow.Price = ProductItem.price;
        NewRow.Amount = ProductItem.quantity * ProductItem.price;
        
    EndDo;
    
    // ─── Jami summa ───
    SalesOrderObj.TotalAmount = SalesOrderObj.Products.Total("Amount");
    
    // ─── Saqlash ───
    SalesOrderObj.Write(DocumentWriteMode.Posting);
    
    Return SalesOrderObj;
    
EndFunction
```

### Handler: Savdo buyurtmasi yaratish

```1c
Function SalesOrderPOST(Request) Export
    
    // ─── JSON parse ───
    RequestBody = Request.GetBodyAsString();
    JSONReader = New JSONReader;
    JSONReader.SetString(RequestBody);
    Data = ReadJSON(JSONReader);
    JSONReader.Close();
    
    // ─── Validatsiya ───
    If Not Data.Property("customer_id") Then
        Return ErrorResponse(400, "customer_id is required");
    EndIf;
    
    If Not Data.Property("products") Then
        Return ErrorResponse(400, "products array is required");
    EndIf;
    
    // ─── Kontragentni topish ───
    CustomerRef = GetCounterpartyByID(Data.customer_id);
    If CustomerRef = Undefined Then
        Return ErrorResponse(404, "Customer not found");
    EndIf;
    
    // ─── Savdo buyurtmasi yaratish ───
    Try
        NewOrder = CreateSalesOrder(CustomerRef, Data.products);
    Except
        Return ErrorResponse(500, "Failed to create order: " + ErrorDescription());
    EndTry;
    
    // ─── Muvaffaqiyatli javob ───
    ResultData = New Structure;
    ResultData.Insert("order_id", String(NewOrder.Ref.UUID()));
    ResultData.Insert("order_number", NewOrder.Number);
    ResultData.Insert("status", "posted");
    ResultData.Insert("total_amount", NewOrder.TotalAmount);
    
    Return SuccessResponse(201, ResultData);
    
EndFunction
```

## 4.6 Xatoliklarni boshqarish

### Try-Except yordamida

```1c
Function SafeHandler(Request) Export
    
    Try
        
        // ─── Asosiy logika ───
        Result = ProcessRequest(Request);
        Return SuccessResponse(200, Result);
        
    Except
        
        // ─── Xatolikni log qilish ───
        ErrorText = ErrorDescription();
        WriteLogEvent(
            "HTTP.Service.Error",
            EventLogLevel.Error,
            ,
            ,
            ErrorText
        );
        
        // ─── Xatolik javobini qaytarish ───
        Return ErrorResponse(500, "Internal server error");
        
    EndTry;
    
EndFunction
```

### Validation funksiyasi

```1c
Function ValidateCounterpartyData(Data)
    
    Errors = New Array;
    
    // ─── Name tekshirish ───
    If Not Data.Property("name") Or IsBlankString(Data.name) Then
        Errors.Add("Name is required");
    EndIf;
    
    // ─── INN tekshirish ───
    If Data.Property("inn") Then
        If StrLen(Data.inn) <> 9 And StrLen(Data.inn) <> 10 Then
            Errors.Add("INN must be 9 or 10 digits");
        EndIf;
    EndIf;
    
    // ─── Email tekshirish ───
    If Data.Property("email") Then
        If Not IsValidEmail(Data.email) Then
            Errors.Add("Invalid email format");
        EndIf;
    EndIf;
    
    Return Errors;
    
EndFunction
```

### Validation'li handler

```1c
Function CounterpartyPOST(Request) Export
    
    // ─── JSON parse ───
    RequestBody = Request.GetBodyAsString();
    JSONReader = New JSONReader;
    JSONReader.SetString(RequestBody);
    Data = ReadJSON(JSONReader);
    JSONReader.Close();
    
    // ─── Validation ───
    ValidationErrors = ValidateCounterpartyData(Data);
    
    If ValidationErrors.Count() > 0 Then
        ErrorData = New Structure;
        ErrorData.Insert("error", True);
        ErrorData.Insert("message", "Validation failed");
        ErrorData.Insert("errors", ValidationErrors);
        Return SuccessResponse(400, ErrorData);
    EndIf;
    
    // ─── Kontragent yaratish ───
    Try
        NewCounterparty = CreateCounterparty(
            Data.name,
            Data.Get("inn"),
            Data.Get("address")
        );
    Except
        Return ErrorResponse(500, ErrorDescription());
    EndTry;
    
    // ─── Muvaffaqiyatli javob ───
    ResultData = New Structure;
    ResultData.Insert("id", String(NewCounterparty.Ref.UUID()));
    ResultData.Insert("status", "created");
    
    Return SuccessResponse(201, ResultData);
    
EndFunction
```

## 4.7 Paginatsiya

Katta ro'yxatlar uchun sahifalash:

```1c
Function CounterpartyGET(Request) Export
    
    // ─── Pagination parametrlari ───
    Page = Number(Request.URLParameters.Get("page"));
    PageSize = Number(Request.URLParameters.Get("page_size"));
    
    If Page = 0 Then Page = 1; EndIf;
    If PageSize = 0 Then PageSize = 20; EndIf;
    
    // ─── Offset hisoblash ───
    Offset = (Page - 1) * PageSize;
    
    // ─── Query ───
    Query = New Query;
    Query.Text = 
    "SELECT
    |    Counterparties.Ref AS Ref,
    |    Counterparties.Description AS Name,
    |    Counterparties.INN AS INN
    |FROM
    |    Catalog.Counterparties AS Counterparties
    |ORDER BY
    |    Counterparties.Description";
    
    Result = Query.Execute();
    Selection = Result.Select();
    
    // ─── Skip va Take ───
    CurrentIndex = 0;
    Items = New Array;
    
    While Selection.Next() Do
        
        CurrentIndex = CurrentIndex + 1;
        
        // Skip
        If CurrentIndex <= Offset Then
            Continue;
        EndIf;
        
        // Take
        If Items.Count() >= PageSize Then
            Break;
        EndIf;
        
        Item = New Structure;
        Item.Insert("id", String(Selection.Ref.UUID()));
        Item.Insert("name", Selection.Name);
        Item.Insert("inn", Selection.INN);
        
        Items.Add(Item);
        
    EndDo;
    
    // ─── Total count ───
    TotalCount = Result.Unload().Count();
    TotalPages = Int(TotalCount / PageSize) + ?(TotalCount % PageSize > 0, 1, 0);
    
    // ─── Response ───
    ResultData = New Structure;
    ResultData.Insert("items", Items);
    ResultData.Insert("page", Page);
    ResultData.Insert("page_size", PageSize);
    ResultData.Insert("total_count", TotalCount);
    ResultData.Insert("total_pages", TotalPages);
    
    Return SuccessResponse(200, ResultData);
    
EndFunction
```

## 4.8 Filterlar

```1c
Function ProductGET(Request) Export
    
    // ─── Filter parametrlari ───
    CategoryFilter = Request.URLParameters.Get("category");
    MinPrice = Request.URLParameters.Get("min_price");
    MaxPrice = Request.URLParameters.Get("max_price");
    SearchQuery = Request.URLParameters.Get("search");
    
    // ─── Query yaratish ───
    Query = New Query;
    QueryText = 
    "SELECT
    |    Products.Ref AS Ref,
    |    Products.Description AS Name,
    |    Products.Code AS Code,
    |    Products.Price AS Price,
    |    Products.Category AS Category
    |FROM
    |    Catalog.Products AS Products
    |WHERE
    |    TRUE";
    
    // ─── Dinamik filterlar ───
    If Not IsBlankString(CategoryFilter) Then
        QueryText = QueryText + "
        |    AND Products.Category = &Category";
        Query.SetParameter("Category", Catalogs.ProductCategories.FindByDescription(CategoryFilter));
    EndIf;
    
    If Not IsBlankString(MinPrice) Then
        QueryText = QueryText + "
        |    AND Products.Price >= &MinPrice";
        Query.SetParameter("MinPrice", Number(MinPrice));
    EndIf;
    
    If Not IsBlankString(MaxPrice) Then
        QueryText = QueryText + "
        |    AND Products.Price <= &MaxPrice";
        Query.SetParameter("MaxPrice", Number(MaxPrice));
    EndIf;
    
    If Not IsBlankString(SearchQuery) Then
        QueryText = QueryText + "
        |    AND (Products.Description LIKE &Search
        |         OR Products.Code LIKE &Search)";
        Query.SetParameter("Search", "%" + SearchQuery + "%");
    EndIf;
    
    Query.Text = QueryText;
    
    // ─── Bajarish va qaytarish ───
    Result = Query.Execute();
    Items = ConvertSelectionToArray(Result.Select());
    
    ResultData = New Structure;
    ResultData.Insert("items", Items);
    ResultData.Insert("count", Items.Count());
    
    Return SuccessResponse(200, ResultData);
    
EndFunction
```

## 4.9 Foydali yordamchi funksiyalar

### GetCounterpartyByID

```1c
Function GetCounterpartyByID(IDString)
    
    Try
        UUID = New UUID(IDString);
        Ref = Catalogs.Counterparties.GetRef(UUID);
        
        If Ref.IsEmpty() Then
            Return Undefined;
        EndIf;
        
        Return Ref;
    Except
        Return Undefined;
    EndTry;
    
EndFunction
```

### ConvertSelectionToArray

```1c
Function ConvertSelectionToArray(Selection)
    
    Items = New Array;
    
    While Selection.Next() Do
        
        Item = New Structure;
        
        For Each Column In Selection.Owner().Columns Do
            Item.Insert(Column.Name, Selection[Column.Name]);
        EndDo;
        
        Items.Add(Item);
        
    EndDo;
    
    Return Items;
    
EndFunction
```

### IsValidEmail

```1c
Function IsValidEmail(Email)
    
    If IsBlankString(Email) Then
        Return False;
    EndIf;
    
    // Oddiy email validatsiya
    Return StrFind(Email, "@") > 0 And StrFind(Email, ".") > 0;
    
EndFunction
```

### WriteLogEvent wrapper

```1c
Procedure LogHTTPRequest(EventName, Data, Level = Undefined)
    
    If Level = Undefined Then
        Level = EventLogLevel.Information;
    EndIf;
    
    JSONWriter = New JSONWriter;
    JSONWriter.SetString();
    WriteJSON(JSONWriter, Data);
    JSONString = JSONWriter.Close();
    
    WriteLogEvent(
        "HTTP.Service." + EventName,
        Level,
        ,
        ,
        JSONString
    );
    
EndProcedure
```

## 4.10 To'liq misol: Mahsulot API

```1c
//═══════════════════════════════════════════════════════════
//# ProductGET - Mahsulotlarni olish
//═══════════════════════════════════════════════════════════

Function ProductGET(Request) Export
    
    Try
        
        // Parametrlar
        ID = Request.URLParameters.Get("id");
        
        If Not IsBlankString(ID) Then
            // Bitta mahsulot
            Product = GetProductByID(ID);
            If Product = Undefined Then
                Return ErrorResponse(404, "Product not found");
            EndIf;
            Return SuccessResponse(200, Product);
        Else
            // Ro'yxat
            Products = GetProductList(Request);
            Return SuccessResponse(200, Products);
        EndIf;
        
    Except
        LogHTTPRequest("Product.GET.Error", ErrorDescription(), EventLogLevel.Error);
        Return ErrorResponse(500, "Internal server error");
    EndTry;
    
EndFunction

//═══════════════════════════════════════════════════════════
//# ProductPOST - Yangi mahsulot yaratish
//═══════════════════════════════════════════════════════════

Function ProductPOST(Request) Export
    
    Try
        
        // JSON parse
        RequestBody = Request.GetBodyAsString();
        JSONReader = New JSONReader;
        JSONReader.SetString(RequestBody);
        Data = ReadJSON(JSONReader);
        JSONReader.Close();
        
        // Validation
        Errors = ValidateProductData(Data);
        If Errors.Count() > 0 Then
            Return ValidationErrorResponse(Errors);
        EndIf;
        
        // Yaratish
        NewProduct = CreateProduct(Data);
        
        // Log
        LogHTTPRequest("Product.Created", String(NewProduct.Ref.UUID()));
        
        // Response
        ResultData = New Structure;
        ResultData.Insert("id", String(NewProduct.Ref.UUID()));
        ResultData.Insert("code", NewProduct.Code);
        ResultData.Insert("status", "created");
        
        Return SuccessResponse(201, ResultData);
        
    Except
        LogHTTPRequest("Product.POST.Error", ErrorDescription(), EventLogLevel.Error);
        Return ErrorResponse(500, "Failed to create product");
    EndTry;
    
EndFunction
```

## 4.11 Best Practices

### ✅ To'g'ri yondashuvlar:

1. **Har doim Try-Except ishlatish**
2. **Validatsiya qo'shish**
3. **To'g'ri HTTP status kodlarini qaytarish**
4. **Loglarni yozish**
5. **UTF-8 encoding ishlatish**
6. **Optimallashtirilgan query'lar yozish**

### ❌ Noto'g'ri yondashuvlar:

1. Xatoliklarni ignore qilish
2. Barcha so'rovlarga 200 qaytarish
3. Validatsiya qilmaslik
4. Xavfsizlikni hisobga olmaslik
5. Katta ma'lumotlarni paginatsiya qilmaslik

## 4.12 Xulosa

Ushbu bo'limda siz:

✅ Request va Response ob'ektlari bilan ishlashni o'rgandingiz  
✅ JSON yaratish va parse qilishni o'rgandingiz  
✅ Ma'lumotlar bazasi bilan ishlashni o'rgandingiz  
✅ Validation va error handling qo'shishni o'rgandingiz  
✅ Pagination va filtering'ni amalga oshirishni o'rgandingiz  

**Navbatdagi bo'lim:** [Odoo bilan integratsiya](05-odoo-integration.md)

---

[← Web Server](03-web-server-setup.md) | [Bosh sahifa](../README.md) | [Odoo Integration →](05-odoo-integration.md)