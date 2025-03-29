
# راهنمای نصب و راه‌اندازی Docker Containers در MikroTik

این دستورالعمل برای نصب و راه‌اندازی Docker Containerها در دستگاه‌های MikroTik با استفاده از نسخه‌های 7.5 به بالا و همچنین راه‌اندازی سرویس‌هایی مانند x-ui است.

## 1. بروزرسانی MikroTik به نسخه 7.5 و بالاتر
ابتدا MikroTik خود را به نسخه 7.5 یا بالاتر بروزرسانی کنید.

## 2. نصب بسته‌های Container از سایت Mikrotik
1. به لینک زیر بروید:
   [MikroTik Archive](https://mikrotik.com/download/archive)
2. فایل‌های مربوط به بسته‌های Container را دانلود کنید.

## 3. بارگذاری فایل‌های Container بر روی MikroTik
1. فایل‌های Container که دانلود کرده‌اید را در بخش File دستگاه MikroTik بارگذاری کنید.
2. دستگاه MikroTik را ریستارت کنید تا نصب Container انجام شود.

## 4. فعال‌سازی Container
پس از ریستارت، دستورات زیر را اجرا کنید تا Container فعال شود:

```bash
/system/device-mode/update container=yes
```

سپس دستگاه را خاموش کرده و دوباره روشن کنید تا تغییرات اعمال شود.

## 5. تنظیمات پیکربندی Container

در ادامه دستورالعمل‌هایی برای تنظیمات پیکربندی Container را انجام دهید:

```bash
/container/config/set ram-high=0 registry-url=https://registry-1.docker.io tmpdir=pull
```

(اگر سرور شما در ایران است از [docker.arvancloud.ir](https://docker.arvancloud.ir) استفاده کنید)

## 6. تنظیمات شبکه

1. ایجاد و تنظیمات veth:

```bash
/interface/veth add name=veth1 address=172.17.0.2/24 gateway=172.17.0.1
/interface bridge add name=dockers
/interface bridge port add bridge=dockers interface=veth1
/ip/address add address=172.17.0.1/24 interface=veth1
```

برای اضافه کردن کانتینرهای بیشتر، از دستورات زیر استفاده کنید:

```bash
/interface/veth add name=veth2 address=172.17.0.3/24 gateway=172.17.0.1
/interface bridge port add bridge=dockers interface=veth2
```

## 7. تنظیمات مربوط به Mounts

```bash
/container/mounts add dst=/etc/x-ui/ name=x-ui src=/x-ui/db
/container/mounts add dst=/root/cert/ name=cert src=/x-ui/cert
```

## 8. اضافه کردن Container جدید

در این بخش می‌توانید کانتینر جدید را با استفاده از دستور زیر راه‌اندازی کنید:

```bash
/container/add interface=veth1 mounts=cert,x-ui workdir=/app start-on-boot=yes remote-image=ghcr.io/mhsanaei/3x-ui:latest
```

## 9. تنظیمات NAT فایروال

در این بخش، قوانین NAT فایروال را اضافه کنید:

```bash
/ip/firewall/nat add chain=srcnat action=masquerade
```

## 10. تنظیمات دایرکتوری NAT برای دسترسی به کانتینرها

برای روتینگ صحیح به کانتینر، IP واقعی MikroTik را دریافت کرده و از آن در دستورات استفاده کنید:

```bash
:global myIP [/ip address get [find interface="ether1"] address]
/ip/firewall/nat add chain=dstnat dst-address=$myIP protocol=tcp dst-port=2053 action=dst-nat to-addresses=172.17.0.2 to-ports=2053
```

> **توجه**: لطفاً مطمئن شوید که در بخش `/ip/firewall/nat`، پورت‌ها به درستی فوروارد شده‌اند و IP درست برای روتر شما تنظیم شده است.
