# راهنمای نصب و راه‌اندازی Filebrowser با Nginx

این راهنما به شما کمک می‌کند تا [Filebrowser](https://filebrowser.org)  را به عنوان یک فایل منیجر ساده و قدرتمند نصب کنید. همچنین از **Nginx** برای ایجاد پروکسی معکوس استفاده خواهیم کرد تا دسترسی به Filebrowser را از طریق یک مسیر خاص (مثل `/filebw`) با **HTTPS** امکان‌پذیر کنیم.

## ۱. نصب Filebrowser

برای نصب Filebrowser، از اسکریپت آماده استفاده می‌کنیم:

```bash
curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash
```

برای بررسی نصب موفقیت‌آمیز، نسخه نصب شده را چک کنید:

```bash
filebrowser version
```

## ۲. تنظیم Filebrowser به عنوان سرویس

برای اینکه Filebrowser به صورت یک سرویس در پس‌زمینه اجرا شود، فایل زیر را ایجاد کنید:

```bash
sudo nano /etc/systemd/system/filebrowser.service
```

متن زیر را در فایل وارد کنید:

```ini
[Unit]
Description=File browser: %I
After=network.target

[Service]
User=www-data
Group=www-data
ExecStart=/usr/local/bin/filebrowser -c /etc/filebrowser/default.json

[Install]
WantedBy=multi-user.target
```

ذخیره کنید: `CTRL+X`، سپس `Y` و `Enter`.

## ۳. تنظیمات Filebrowser

### ۳.۱ ایجاد دایرکتوری تنظیمات

یک دایرکتوری برای ذخیره تنظیمات Filebrowser ایجاد کنید:

```bash
sudo mkdir /etc/filebrowser
```

### ۳.۲ ایجاد فایل پیکربندی

یک فایل پیکربندی جدید به نام `default.json` ایجاد کنید:

```bash
sudo nano /etc/filebrowser/default.json
```

محتوای زیر را وارد کنید:

```json
{
  "port": 8080,
  "baseURL": "/filebw",
  "address": "127.0.0.1",
  "log": "stdout",
  "database": "/etc/filebrowser/filebrowser.db",
  "root": "/path/to/your/files"
}
```

**نکته:** مقدار `/path/to/your/files` را با مسیر دایرکتوری‌ای که می‌خواهید Filebrowser مدیریت کند جایگزین کنید.

### ۳.۳ ایجاد پایگاه داده

پایگاه داده را برای Filebrowser ایجاد کنید:

```bash
filebrowser -d /etc/filebrowser/filebrowser.db
```

### ۳.۴ تنظیم دسترسی‌ها

دسترسی‌های لازم را تنظیم کنید:

```bash
sudo chown -R www-data:www-data /etc/filebrowser/filebrowser.db
```

## ۴. فعال‌سازی و اجرای سرویس

### ۴.۱ فعال‌سازی سرویس در بوت

ابتدا Filebrowser را طوری تنظیم کنید که با روشن شدن سرور اجرا شود:

```bash
sudo systemctl enable filebrowser
```

### ۴.۲ شروع سرویس

سرویس را شروع کنید:

```bash
sudo systemctl start filebrowser
```

### ۴.۳ بررسی وضعیت سرویس

وضعیت سرویس را بررسی کنید:

```bash
sudo systemctl status filebrowser
```

## ۵. تنظیمات Nginx

فرض می‌کنیم **Nginx** قبلا نصب شده است. فایل تنظیمات Nginx مربوط به دامنه خود را باز کنید:

```bash
sudo nano /etc/nginx/sites-available/yourdomain
```

بخش زیر را برای مسیر `/filebw` اضافه کنید:

```nginx
location /filebw/ {
    proxy_pass http://127.0.0.1:8080/;  # آدرس سرور Filebrowser
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    rewrite ^/filebw(/.*)$ $1 break;  # بازنویسی مسیر
}
```

**نکات:**
- **فایل تنظیمات Nginx:** اطمینان حاصل کنید که فایل تنظیمات Nginx مربوط به دامنه شما به درستی تنظیم شده و سایر تنظیمات مورد نیاز نیز اعمال شده باشند.
- **پورت Filebrowser:** مطمئن شوید که پورت تعیین‌شده در فایل پیکربندی Filebrowser (`8080` در اینجا) با پورت استفاده‌شده در `proxy_pass` مطابقت دارد.

## ۶. تست و ری‌استارت Nginx

سرویس Nginx را ری‌استارت کنید:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## ۷. تست و دسترسی به Filebrowser

پس از تنظیمات، می‌توانید Filebrowser را در مسیر زیر مشاهده کنید:

```bash
https://yourdomain/filebw
```

### ۷.۱ اطلاعات ورود پیش‌فرض

- **نام کاربری:** `admin`
- **رمز عبور:** `admin`

### ۷.۲ تغییر رمز عبور

پس از ورود، رمز عبور را تغییر دهید تا امنیت بیشتری داشته باشید.!
