# Snort 3 Windows Integration

## معرفی
این مخزن یک پروژه‌ی Integration برای اجرای **Snort 3 روی Windows** با استفاده از **LibDAQ** و **Npcap** است. هدف این است که Windows support در دو پروژه‌ی upstream به‌صورت مستقل توسعه داده شود و سپس نسخه‌های مربوطه در این مخزن کنار هم قرار بگیرند تا build، تست و بازتولید محیط ساده باشد.

## ساختار پروژه
```text
snort3-windows/
├── snort3/          # Git submodule؛ Snort با Windows support
├── libdaq/          # Git submodule؛ LibDAQ با Windows support
├── docs/            # مستندات فنی، معماری و نتایج تست
├── scripts/         # اسکریپت‌های build و test
├── patches/         # patchهای مستقل، در صورت نیاز
├── rules/           # قوانین کوچک و تستی Snort
├── .gitmodules
├── .gitignore
├── README.md        # مستندات انگلیسی
└── README.fa.md     # مستندات فارسی
```

## اجزای اصلی

### Snort 3
Snort به‌صورت submodule در `snort3/` قرار گرفته و به fork دارای Windows support متصل است.

Upstream PR:
- **snort3/snort3 — PR #478**
- `Add Windows support for Snort`

### LibDAQ
LibDAQ نیز به‌صورت submodule در `libdaq/` قرار گرفته است.

Upstream PR:
- **snort3/libdaq — PR #43**
- `Add Windows support for LibDAQ`

این دو تغییر مستقل از یکدیگر هستند و این مخزن فقط آن‌ها را برای استفاده و تست یکپارچه کنار هم قرار می‌دهد.

## چرا source و install داخل مخزن نیستند؟
پوشه‌ها و فایل‌های تولیدشده یا وابسته به سیستم محلی عمداً commit نمی‌شوند؛ از جمله:

- `build/`
- `install/`
- سورس‌های کپی‌شده‌ی Snort و LibDAQ
- `.exe` و `.dll`
- `.a` و `.o`
- logها
- فایل‌های capture مانند `.pcap`

سورس اصلی از طریق Git submodule دریافت می‌شود و commit ثبت‌شده‌ی submodule نسخه‌ی دقیق مورد استفاده را مشخص می‌کند.

## قوانین Snort
برخلاف log و فایل‌های خروجی، یک Rule کوچک تستی ارزش نگهداری در مخزن دارد، چون بازتولید تست را ممکن می‌کند.

برای نمونه:
```text
rules/windows-test.rules
```

```text
alert icmp any any -> any any (msg:"ICMP ECHO REQUEST"; itype:8; sid:1000001; rev:5;)
```

اطلاعات خصوصی محیط تست، مانند IP داخلی، Gateway واقعی یا UUID مربوط به Npcap، نباید در مخزن عمومی قرار بگیرد.

## نقش پوشه‌ها

### `docs/`
مستندات پروژه در اینجا قرار می‌گیرند؛ برای مثال:
- `build.md` — مراحل build
- `testing.md` — روش تست و نتایج
- `architecture.md` — معماری Snort، LibDAQ و Npcap

### `scripts/`
برای خودکار کردن کارهای تکراری است؛ مثل build کردن LibDAQ، build کردن Snort و اجرای تست‌های شبکه. مسیرهای شخصی سیستم نباید hard-code شوند.

### `patches/`
برای نگهداری patch fileهای مستقل در صورت نیاز است. چون تغییرات اصلی فعلاً در forkهای GitHub نگهداری می‌شوند، لازم نیست source یا patch تکراری داخل این پوشه قرار بگیرد. در وضعیت فعلی می‌تواند خالی بماند.

## معماری
```text
Windows
   │
   ├── Npcap
   │     │
   │     ▼
   │   LibDAQ
   │     │
   │     ▼
   │   Snort 3
   │     │
   │     ├── Rules
   │     └── Alerts / Logs
   │
   └── Test Traffic
```

Npcap ترافیک شبکه را دریافت می‌کند، LibDAQ دسترسی Snort به capture را فراهم می‌کند و Snort بسته‌ها را با Ruleها پردازش می‌کند.

## محیط تست
توسعه و validation پروژه روی این محیط انجام شده است:
- Windows 10 Enterprise 64-bit
- MSYS2 UCRT64
- Snort 3.12.2.0
- LibDAQ 3.0.27
- Npcap 1.88

در تست عملی، Snort توانست ترافیک زنده را از طریق Npcap دریافت کند، DAQ pcap را بارگذاری کند و Rule تشخیص ICMP را روی ترافیک واقعی فعال کند.

## دریافت پروژه
```bash
git clone https://github.com/Mahbodbe/snort3-windows.git
cd snort3-windows
git submodule update --init --recursive
```

در Git submodule، **commit ثبت‌شده در مخزن Integration** نسخه‌ی واقعی مورد استفاده را مشخص می‌کند؛ مقدار `branch` در `.gitmodules` صرفاً شاخه‌ی پیگیری‌شده برای update است.

## وضعیت Upstream
Windows support به‌صورت دو Pull Request مستقل به پروژه‌های اصلی پیشنهاد شده است:
1. Snort 3 — PR #478
2. LibDAQ — PR #43

تا زمان merge شدن این PRها، Integration Repository از forkهای دارای Windows support استفاده می‌کند.

## خلاصه
این پروژه سه بخش را از هم جدا می‌کند:
1. Windows support برای Snort
2. Windows support برای LibDAQ
3. Integration و validation این دو در محیط Windows

این ساختار باعث می‌شود تغییرات اصلی مستقل و قابل بررسی باشند و در عین حال یک مخزن تمیز و قابل بازتولید برای build و تست وجود داشته باشد.
