# Odoo bilan integratsiya

## 5.1 Umumiy ma'lumot

Odoo - bu ochiq manba ERP tizimi bo'lib, Python dasturlash tilida yozilgan. 1C bilan integratsiya qilish uchun Odoo'dan HTTP so'rovlarni yuboramiz.

### Integratsiya sxemasi:

```
┌──────────────────┐                      ┌──────────────────┐
│                  │   HTTP GET/POST      │                  │
│   Odoo (Python)  ├─────────────────────►│  1C:Enterprise   │
│                  │   JSON Request       │                  │
│                  │                      │  HTTP Service    │
│                  │   JSON Response      │                  │
│                  │◄─────────────────────┤  Handler Module  │
│                  │                      │                  │
└──────────────────┘                      └──────────────────┘
```

## 5.2 Python kutubxonalari

### requests kutubxonasi

Eng oddiy va mashhur HTTP kutubxona:

```bash
pip install requests
```

### Asosiy GET so'rov:

```python
import requests

url = "http://localhost/radchenko/hs/exchange/test"
response = requests.get(url)

print(f"Status Code: {response.status_code}")
print(f"Response: {response.text}")
```

### Asosiy POST so'rov:

```python
import requests
import json

url = "http://localhost/radchenko/hs/exchange/counterparty"

data = {
    "name": "New Counterparty",
    "inn": "1234567890",
    "address": "Tashkent, Uzbekistan"
}

headers = {
    "Content-Type": "application/json"
}

response = requests.post(url, json=data, headers=headers)

print(f"Status Code: {response.status_code}")
print(f"Response: {response.json()}")
```

## 5.3 Odoo model yaratish

### Qadam 1: Yangi modul yaratish

Odoo modulida 1C integratsiyasi uchun model yaratamiz:

```
my_module/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── integration_1c.py
└── views/
    └── integration_1c_views.xml
```

### Qadam 2: `__manifest__.py`

```python
{
    'name': '1C Integration',
    'version': '1.0',
    'category': 'Integration',
    'summary': 'Integration with 1C:Enterprise',
    'description': """
        This module provides integration with 1C:Enterprise
        via HTTP services.
    """,
    'depends': ['base', 'sale', 'purchase'],
    'data': [
        'views/integration_1c_views.xml',
    ],
    'installable': True,
    'application': False,
}
```

### Qadam 3: `models/integration_1c.py`

```python
from odoo import models, fields, api
import requests
import json
import logging

_logger = logging.getLogger(__name__)


class Integration1C(models.Model):
    _name = 'integration.1c'
    _description = '1C Integration'
    
    # ═══════════════════════════════════════════════════════
    # FIELDS
    # ═══════════════════════════════════════════════════════
    
    name = fields.Char('Name', required=True)
    base_url = fields.Char('Base URL', required=True, 
                           default='http://localhost/radchenko/hs/exchange')
    username = fields.Char('Username')
    password = fields.Char('Password')
    active = fields.Boolean('Active', default=True)
    
    # ═══════════════════════════════════════════════════════
    # HELPER METHODS
    # ═══════════════════════════════════════════════════════
    
    def _get_headers(self):
        """Return HTTP headers for requests"""
        return {
            'Content-Type': 'application/json; charset=utf-8',
            'Accept': 'application/json'
        }
    
    def _get_auth(self):
        """Return authentication tuple if configured"""
        if self.username and self.password:
            return (self.username, self.password)
        return None
    
    def _make_request(self, method, endpoint, data=None, params=None):
        """
        Make HTTP request to 1C
        
        :param method: HTTP method (GET, POST, etc.)
        :param endpoint: API endpoint (e.g., '/counterparty')
        :param data: Request body (dict)
        :param params: URL parameters (dict)
        :return: Response object or None
        """
        url = f"{self.base_url}{endpoint}"
        
        try:
            response = requests.request(
                method=method,
                url=url,
                json=data,
                params=params,
                headers=self._get_headers(),
                auth=self._get_auth(),
                timeout=30
            )
            response.raise_for_status()
            return response
            
        except requests.exceptions.RequestException as e:
            _logger.error(f"1C Request Error: {e}")
            raise
    
    # ═══════════════════════════════════════════════════════
    # API METHODS
    # ═══════════════════════════════════════════════════════
    
    def get_counterparties(self, active=True):
        """Get counterparty list from 1C"""
        params = {'active': 'true' if active else 'false'}
        response = self._make_request('GET', '/counterparty', params=params)
        return response.json()
    
    def create_counterparty(self, name, inn, address):
        """Create new counterparty in 1C"""
        data = {
            'name': name,
            'inn': inn,
            'address': address
        }
        response = self._make_request('POST', '/counterparty', data=data)
        return response.json()
    
    def get_products(self, category=None, min_price=None, max_price=None):
        """Get product list from 1C"""
        params = {}
        if category:
            params['category'] = category
        if min_price:
            params['min_price'] = min_price
        if max_price:
            params['max_price'] = max_price
            
        response = self._make_request('GET', '/product', params=params)
        return response.json()
    
    def create_sales_order(self, customer_id, products):
        """
        Create sales order in 1C
        
        :param customer_id: 1C Counterparty UUID
        :param products: List of dicts with keys: code, quantity, price
        """
        data = {
            'customer_id': customer_id,
            'products': products
        }
        response = self._make_request('POST', '/salesorder', data=data)
        return response.json()
    
    def get_stock(self, product_code=None):
        """Get stock information from 1C"""
        params = {}
        if product_code:
            params['product_code'] = product_code
            
        response = self._make_request('GET', '/stock', params=params)
        return response.json()
```

## 5.4 Odoo'dan 1C'ga ma'lumot yuborish

### Kontragentni sinxronlash

Odoo'da yangi kontragent yaratilganda uni 1C'ga ham yuborish:

```python
from odoo import models, fields, api


class ResPartner(models.Model):
    _inherit = 'res.partner'
    
    # Yangi maydon: 1C ID
    x_1c_id = fields.Char('1C ID', readonly=True)
    x_sync_to_1c = fields.Boolean('Sync to 1C', default=False)
    
    @api.model
    def create(self, vals):
        """Override create to sync with 1C"""
        partner = super(ResPartner, self).create(vals)
        
        # Agar sync yoqilgan bo'lsa
        if partner.x_sync_to_1c:
            partner._sync_to_1c()
        
        return partner
    
    def write(self, vals):
        """Override write to sync with 1C"""
        result = super(ResPartner, self).write(vals)
        
        # Agar sync yoqilgan bo'lsa
        for partner in self:
            if partner.x_sync_to_1c and partner.x_1c_id:
                partner._update_in_1c()
        
        return result
    
    def _sync_to_1c(self):
        """Send partner data to 1C"""
        integration = self.env['integration.1c'].search([('active', '=', True)], limit=1)
        
        if not integration:
            raise UserError('1C Integration not configured')
        
        try:
            # 1C'ga yuborish
            result = integration.create_counterparty(
                name=self.name,
                inn=self.vat or '',
                address=self._format_address()
            )
            
            # 1C ID saqlash
            if result.get('id'):
                self.x_1c_id = result['id']
                _logger.info(f"Partner {self.name} synced to 1C: {self.x_1c_id}")
        
        except Exception as e:
            _logger.error(f"Failed to sync partner to 1C: {e}")
            raise
    
    def _format_address(self):
        """Format address for 1C"""
        parts = []
        if self.street:
            parts.append(self.street)
        if self.street2:
            parts.append(self.street2)
        if self.city:
            parts.append(self.city)
        if self.state_id:
            parts.append(self.state_id.name)
        if self.country_id:
            parts.append(self.country_id.name)
        return ', '.join(parts)
```

### Savdo buyurtmasini yuborish

```python
from odoo import models, api


class SaleOrder(models.Model):
    _inherit = 'sale.order'
    
    x_1c_order_id = fields.Char('1C Order ID', readonly=True)
    x_1c_order_number = fields.Char('1C Order Number', readonly=True)
    
    def action_confirm(self):
        """Override confirm to send to 1C"""
        result = super(SaleOrder, self).action_confirm()
        
        # Tasdiqdan keyin 1C'ga yuborish
        for order in self:
            if order.partner_id.x_sync_to_1c:
                order._send_to_1c()
        
        return result
    
    def _send_to_1c(self):
        """Send sale order to 1C"""
        integration = self.env['integration.1c'].search([('active', '=', True)], limit=1)
        
        if not integration:
            return
        
        # Mijoz 1C ID
        customer_1c_id = self.partner_id.x_1c_id
        if not customer_1c_id:
            _logger.warning(f"Partner {self.partner_id.name} has no 1C ID")
            return
        
        # Mahsulotlar ro'yxati
        products = []
        for line in self.order_line:
            products.append({
                'code': line.product_id.default_code or '',
                'quantity': line.product_uom_qty,
                'price': line.price_unit
            })
        
        try:
            # 1C'ga yuborish
            result = integration.create_sales_order(customer_1c_id, products)
            
            # 1C ma'lumotlarini saqlash
            if result.get('order_id'):
                self.x_1c_order_id = result['order_id']
            if result.get('order_number'):
                self.x_1c_order_number = result['order_number']
            
            _logger.info(f"Sale Order {self.name} sent to 1C: {self.x_1c_order_number}")
        
        except Exception as e:
            _logger.error(f"Failed to send order to 1C: {e}")
```

## 5.5 1C'dan Odoo'ga ma'lumot olish

### Mahsulotlar ro'yxatini import qilish

```python
from odoo import models, api
from odoo.exceptions import UserError


class ProductTemplate(models.Model):
    _inherit = 'product.template'
    
    def action_import_from_1c(self):
        """Import products from 1C"""
        integration = self.env['integration.1c'].search([('active', '=', True)], limit=1)
        
        if not integration:
            raise UserError('1C Integration not configured')
        
        try:
            # 1C'dan mahsulotlarni olish
            result = integration.get_products()
            products_data = result.get('items', [])
            
            imported_count = 0
            updated_count = 0
            
            for product_data in products_data:
                product = self._find_or_create_product(product_data)
                if product:
                    if product.id:
                        updated_count += 1
                    else:
                        imported_count += 1
            
            message = f"Import completed: {imported_count} new, {updated_count} updated"
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'title': '1C Import',
                    'message': message,
                    'type': 'success',
                    'sticky': False,
                }
            }
        
        except Exception as e:
            raise UserError(f"Import failed: {str(e)}")
    
    def _find_or_create_product(self, data):
        """Find existing product or create new one"""
        # 1C kodiga qarab qidirish
        product_code = data.get('code')
        product = self.search([('default_code', '=', product_code)], limit=1)
        
        vals = {
            'name': data.get('name'),
            'default_code': product_code,
            'list_price': data.get('price', 0),
            'type': 'product',
        }
        
        if product:
            # Yangilash
            product.write(vals)
        else:
            # Yaratish
            product = self.create(vals)
        
        return product
```

### Ombor qoldig'ini tekshirish

```python
from odoo import models, fields, api


class ProductProduct(models.Model):
    _inherit = 'product.product'
    
    x_1c_stock = fields.Float('1C Stock', compute='_compute_1c_stock')
    
    def _compute_1c_stock(self):
        """Get stock from 1C"""
        integration = self.env['integration.1c'].search([('active', '=', True)], limit=1)
        
        for product in self:
            product.x_1c_stock = 0
            
            if not integration or not product.default_code:
                continue
            
            try:
                result = integration.get_stock(product_code=product.default_code)
                stock_data = result.get('items', [])
                
                if stock_data:
                    product.x_1c_stock = stock_data[0].get('quantity', 0)
            
            except Exception as e:
                _logger.error(f"Failed to get stock from 1C: {e}")
```

## 5.6 Scheduled Actions (Cron)

Avtomatik sinxronlash uchun cron yaratamiz.

### XML fayl: `data/cron.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        
        <!-- Mahsulotlarni har kuni import qilish -->
        <record id="cron_import_products_from_1c" model="ir.cron">
            <field name="name">Import Products from 1C</field>
            <field name="model_id" ref="product.model_product_template"/>
            <field name="state">code</field>
            <field name="code">model.action_import_from_1c()</field>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
            <field name="active">True</field>
        </record>
        
        <!-- Ombor qoldig'ini har soatda yangilash -->
        <record id="cron_sync_stock_from_1c" model="ir.cron">
            <field name="name">Sync Stock from 1C</field>
            <field name="model_id" ref="product.model_product_product"/>
            <field name="state">code</field>
            <field name="code">model.action_sync_stock_from_1c()</field>
            <field name="interval_number">1</field>
            <field name="interval_type">hours</field>
            <field name="numbercall">-1</field>
            <field name="active">True</field>
        </record>
        
    </data>
</odoo>
```

## 5.7 Error Handling va Logging

### Logging sozlash

```python
import logging

_logger = logging.getLogger(__name__)

# Different levels
_logger.debug("Debug message")
_logger.info("Info message")
_logger.warning("Warning message")
_logger.error("Error message")
_logger.critical("Critical message")
```

### Try-Except bilan xatoliklarni ushlash

```python
def _sync_to_1c(self):
    """Send data to 1C with error handling"""
    integration = self.env['integration.1c'].search([('active', '=', True)], limit=1)
    
    if not integration:
        raise UserError('1C Integration not configured')
    
    try:
        result = integration.create_counterparty(
            name=self.name,
            inn=self.vat or '',
            address=self._format_address()
        )
        
        if result.get('id'):
            self.x_1c_id = result['id']
            self.message_post(body=f"Successfully synced to 1C: {self.x_1c_id}")
        
    except requests.exceptions.Timeout:
        error_msg = "1C server timeout"
        _logger.error(error_msg)
        self.message_post(body=f"Sync failed: {error_msg}")
        raise UserError(error_msg)
    
    except requests.exceptions.ConnectionError:
        error_msg = "Cannot connect to 1C server"
        _logger.error(error_msg)
        self.message_post(body=f"Sync failed: {error_msg}")
        raise UserError(error_msg)
    
    except requests.exceptions.HTTPError as e:
        error_msg = f"1C HTTP error: {e.response.status_code}"
        _logger.error(f"{error_msg} - {e.response.text}")
        self.message_post(body=f"Sync failed: {error_msg}")
        raise UserError(error_msg)
    
    except Exception as e:
        error_msg = f"Unexpected error: {str(e)}"
        _logger.exception(error_msg)
        self.message_post(body=f"Sync failed: {error_msg}")
        raise UserError(error_msg)
```

## 5.8 Webhook (1C → Odoo)

1C'dan Odoo'ga webhook orqali ma'lumot yuborish.

### Odoo controller yaratish

```python
from odoo import http
from odoo.http import request
import json
import logging

_logger = logging.getLogger(__name__)


class OneCWebhook(http.Controller):
    
    @http.route('/api/1c/webhook/order', type='json', auth='public', methods=['POST'], csrf=False)
    def receive_order_from_1c(self, **kwargs):
        """
        Receive order data from 1C
        
        Expected JSON:
        {
            "order_id": "uuid",
            "order_number": "SO-00001",
            "status": "posted",
            "customer_inn": "1234567890",
            "products": [...]
        }
        """
        try:
            data = request.jsonrequest
            
            _logger.info(f"Received webhook from 1C: {data}")
            
            # Mijozni topish
            customer = request.env['res.partner'].sudo().search([
                ('vat', '=', data.get('customer_inn'))
            ], limit=1)
            
            if not customer:
                return {'error': 'Customer not found', 'code': 404}
            
            # Buyurtma yaratish
            order_vals = {
                'partner_id': customer.id,
                'x_1c_order_id': data.get('order_id'),
                'x_1c_order_number': data.get('order_number'),
            }
            
            # Mahsulotlar qo'shish
            order_lines = []
            for product_data in data.get('products', []):
                product = request.env['product.product'].sudo().search([
                    ('default_code', '=', product_data.get('code'))
                ], limit=1)
                
                if product:
                    order_lines.append((0, 0, {
                        'product_id': product.id,
                        'product_uom_qty': product_data.get('quantity'),
                        'price_unit': product_data.get('price'),
                    }))
            
            order_vals['order_line'] = order_lines
            
            # Saqlash
            order = request.env['sale.order'].sudo().create(order_vals)
            
            return {
                'status': 'success',
                'odoo_order_id': order.id,
                'odoo_order_name': order.name
            }
        
        except Exception as e:
            _logger.exception("Webhook error")
            return {'error': str(e), 'code': 500}
```

### 1C'dan webhook yuborish

```1c
Function SendOrderToOdoo(OrderRef)
    
    // ─── Ma'lumotlarni tayyorlash ───
    OrderData = New Structure;
    OrderData.Insert("order_id", String(OrderRef.UUID()));
    OrderData.Insert("order_number", OrderRef.Number);
    OrderData.Insert("status", "posted");
    OrderData.Insert("customer_inn", OrderRef.Customer.INN);
    
    // Mahsulotlar
    ProductsArray = New Array;
    For Each Line In OrderRef.Products Do
        ProductItem = New Structure;
        ProductItem.Insert("code", Line.Product.Code);
        ProductItem.Insert("quantity", Line.Quantity);
        ProductItem.Insert("price", Line.Price);
        ProductsArray.Add(ProductItem);
    EndDo;
    OrderData.Insert("products", ProductsArray);
    
    // ─── JSON yaratish ───
    JSONWriter = New JSONWriter;
    JSONWriter.SetString();
    WriteJSON(JSONWriter, OrderData);
    JSONString = JSONWriter.Close();
    
    // ─── HTTP so'rov yuborish ───
    Connection = New HTTPConnection("odoo.example.com", 80);
    Request = New HTTPRequest("/api/1c/webhook/order");
    Request.Headers.Insert("Content-Type", "application/json");
    Request.SetBodyFromString(JSONString);
    
    Response = Connection.Post(Request);
    
    If Response.StatusCode = 200 Then
        // Muvaffaqiyatli
        ResponseBody = Response.GetBodyAsString();
        _logger.Info("Order sent to Odoo: " + ResponseBody);
    Else
        // Xatolik
        Raise "Failed to send order to Odoo: " + Response.StatusCode;
    EndIf;
    
EndFunction
```

## 5.9 Best Practices

### ✅ Tavsiyalar:

1. **Timeout qo'yish** - Uzoq kutmasin
2. **Retry mexanizmi** - Xatolikda qayta urinish
3. **Queue ishlatish** - Katta ma'lumotlar uchun
4. **Loglarni yozish** - Har bir so'rovni log qilish
5. **Error handling** - Barcha xatoliklarni ushlash
6. **Optimizatsiya** - Batch operatsiyalardan foydalanish

### ❌ Xatolar:

1. Timeoutsiz so'rov yuborish
2. Xatoliklarni ignore qilish
3. Loglarni yozmaslik
4. Transaksiyalarni hisobga olmaslik
5. Katta ma'lumotlarni bir vaqtda yuborish

## 5.10 Xulosa

Ushbu bo'limda siz:

✅ Odoo'da 1C integratsiya modulini yaratishni o'rgandingiz  
✅ Odoo'dan 1C'ga ma'lumot yuborishni o'rgandingiz  
✅ 1C'dan Odoo'ga ma'lumot olishni o'rgandingiz  
✅ Webhook orqali real-time integratsiyani o'rgandingiz  
✅ Error handling va logging'ni qo'shishni o'rgandingiz  

**Navbatdagi bo'lim:** [Autentifikatsiya](06-authentication.md)

---

[← 1C Kod](04-1c-code.md) | [Bosh sahifa](../README.md) | [Authentication →](06-authentication.md)