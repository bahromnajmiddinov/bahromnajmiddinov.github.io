# GitHub Pages Setup qo'llanmasi

Bu faylda dokumentatsiyani GitHub Pages'da qanday e'lon qilish haqida to'liq ma'lumot berilgan.

## 1. GitHub Repository yaratish

### Qadam 1: GitHub'da repository yaratish

1. GitHub'ga kiring: https://github.com
2. O'ng yuqori burchakda **+** belgisini bosing va **New repository** ni tanlang
3. Repository sozlamalari:
   - **Repository name:** `1c-odoo-integration-docs` (yoki istalgan nom)
   - **Description:** `1C va Odoo integratsiya qo'llanmasi`
   - **Public** yoki **Private** tanlang
   - **Initialize with README** - belgisiz qoldiring
4. **Create repository** tugmasini bosing

### Qadam 2: Repository'ni local mashinaga clone qiling

```bash
git clone https://github.com/yourusername/1c-odoo-integration-docs.git
cd 1c-odoo-integration-docs
```

## 2. Dokumentatsiya fayllarini joylashtirish

### Barcha fayllarni copy qilish

Ushbu qo'llanma fayllarini repository papkasiga ko'chiring:

```
1c-odoo-integration-docs/
├── README.md
├── _config.yml
├── .gitignore
├── docs/
│   ├── 01-installation.md
│   ├── 02-http-services.md
│   ├── 03-web-server-setup.md
│   ├── 04-1c-code.md
│   ├── 05-odoo-integration.md
│   ├── 06-authentication.md
│   └── 07-examples.md
└── images/
    └── (rasmlar bu yerga)
```

## 3. Rasmlarni qo'shish

### Rasmlarni tayyorlash

1. Barcha screenshot va diagrammalarni `images/` papkasiga joylashtiring
2. Rasmlarni to'g'ri nomlang:
   - `01-download-page.png`
   - `02-startup-window.png`
   - `03-create-infobase.png`
   - va hokazo...

### Rasmlarni dokumentatsiyada ishlatish

Markdown fayllaridagi rasmlar yo'llari quyidagicha bo'lishi kerak:

```markdown
![1C Download Page](../images/01-download-page.png)
```

## 4. GitHub'ga push qilish

```bash
# Barcha fayllarni staging'ga qo'shish
git add .

# Commit yaratish
git commit -m "Initial documentation commit"

# GitHub'ga push qilish
git push origin main
```

## 5. GitHub Pages'ni yoqish

### Qadam 1: Repository Settings

1. GitHub repository sahifasida **Settings** ga o'ting
2. Chap sidebar'da **Pages** bo'limini toping

### Qadam 2: Source sozlash

1. **Source** bo'limida:
   - **Branch:** `main` (yoki `master`) ni tanlang
   - **Folder:** `/ (root)` ni tanlang
2. **Save** tugmasini bosing

### Qadam 3: Theme tanlash (ixtiyoriy)

1. **Theme Chooser** bo'limida **Choose a theme** ni bosing
2. Yoqqan theme'ni tanlang (yoki `_config.yml`dagi Cayman theme'dan foydalaning)
3. **Select theme** ni bosing

**Eslatma:** `_config.yml` faylida allaqachon Cayman theme o'rnatilgan.

## 6. Site'ni ochish

### URL

GitHub Pages site'ingiz quyidagi URL'da ochiladi:

```
https://yourusername.github.io/1c-odoo-integration-docs/
```

**Masalan:**
- Agar username: `johndoe`
- Repo nomi: `1c-odoo-integration-docs`
- URL: `https://johndoe.github.io/1c-odoo-integration-docs/`

### Kutish

Site build qilinishi 1-2 daqiqa davom etishi mumkin. Shundan keyin yuqoridagi URL'da ochiladi.

## 7. Custom Domain (ixtiyoriy)

Agar o'z domeningiz bo'lsa (masalan: `docs.mycompany.com`):

### Qadam 1: DNS sozlash

Domain provideringizda CNAME record qo'shing:

```
Type: CNAME
Name: docs (yoki subdomain)
Value: yourusername.github.io
```

### Qadam 2: GitHub'da custom domain qo'shish

1. Repository Settings → Pages
2. **Custom domain** bo'limida domenni kiriting: `docs.mycompany.com`
3. **Save** ni bosing

### Qadam 3: HTTPS yoqish

1. **Enforce HTTPS** checkboxni belgilang
2. GitHub avtomatik Let's Encrypt sertifikati o'rnatadi

## 8. Yangilanishlar qo'shish

Dokumentatsiyani yangilash uchun:

```bash
# Fayllarni o'zgartiring
# Keyin:

git add .
git commit -m "Updated documentation"
git push origin main
```

GitHub Pages avtomatik ravishda yangi versiyani publish qiladi (1-2 daqiqa ichida).

## 9. Local'da test qilish (ixtiyoriy)

Lokalda GitHub Pages'ni test qilish uchun Jekyll o'rnatishingiz mumkin:

### Ruby va Jekyll o'rnatish

**Windows:**
1. [RubyInstaller](https://rubyinstaller.org/) yuklab oling
2. Install qiling (with DevKit)
3. Command Prompt'da:

```bash
gem install jekyll bundler
```

**macOS/Linux:**

```bash
# macOS (Homebrew)
brew install ruby
gem install jekyll bundler

# Ubuntu/Debian
sudo apt-get install ruby-full build-essential
gem install jekyll bundler
```

### Local server ishga tushirish

```bash
cd 1c-odoo-integration-docs

# Birinchi marta:
bundle init
bundle add jekyll

# Har safar:
bundle exec jekyll serve
```

Site `http://localhost:4000` da ochiladi.

## 10. Troubleshooting

### Muammo: "404 Page not found"

**Yechim:**
- GitHub Pages yoqilganligini tekshiring (Settings → Pages)
- URL to'g'ri ekanligini tekshiring
- Build tugashini kuting (1-2 daqiqa)

### Muammo: Rasmlar ko'rinmayapti

**Yechim:**
- Rasm yo'llari to'g'ri ekanligini tekshiring
- `../images/filename.png` formatdan foydalaning
- Rasm fayl nomlari kichik harfda va bo'sh joysiz bo'lsin

### Muammo: "Build failed"

**Yechim:**
- Repository Settings → Pages → View build logs
- Syntax xatolarini tekshiring
- `_config.yml` faylni tekshiring

### Muammo: Theme ishlamayapti

**Yechim:**
- `_config.yml` faylda theme to'g'ri yozilganligini tekshiring
- GitHub qo'llab-quvvatlaydigan theme'lardan foydalaning
- Supported themes: https://pages.github.com/themes/

## 11. Qo'shimcha sozlamalar

### Google Analytics qo'shish

`_config.yml` faylida:

```yaml
google_analytics: UA-XXXXXXXXX-X
```

### SEO optimallashtirish

`_config.yml` faylida:

```yaml
title: 1C va Odoo Integratsiya Qo'llanmasi
description: To'liq qo'llanma 1C:Enterprise va Odoo integratsiyasi bo'yicha
lang: uz
```

### Social media tags

README.md faylining boshiga:

```markdown
---
title: 1C va Odoo Integratsiya
description: HTTP servislari orqali integratsiya qo'llanmasi
image: /images/og-image.png
---
```

## 12. Ko'proq ma'lumot

- **GitHub Pages Docs:** https://docs.github.com/en/pages
- **Jekyll Documentation:** https://jekyllrb.com/docs/
- **Markdown Guide:** https://www.markdownguide.org/

---

Muvaffaqiyatlar! 🚀