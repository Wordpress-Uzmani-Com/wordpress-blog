---
title: "WordPress Dosya ve Dizin İzinleri: 644 ve 755 Standartları"
slug: "wordpress-dosya-ve-dizin-izinleri-chmod-standartlari"
date: "2026-11-25 09:30:01"
category: "WordPress Güvenliği"
canonical_url: "https://wordpressuzmani.com/blog/wordpress-dosya-ve-dizin-izinleri-chmod-standartlari"
cover_image: "https://wordpressuzmani.com/uploads/blog/wordpress-dosya-ve-dizin-izinleri-chmod-standartlari.jpg"
read_time: "3 dk"
tags: ["wordpress-güvenliği","644-ve-755"]
description: "WordPress dosya ve dizin izinleri: 644 ve 755 standartlarıyla yetkisiz dosya yazma açıklarını kapatan güvenli chmod matrisi."
---

# WordPress Dosya ve Dizin İzinleri: 644 ve 755 Standartları

**WordPress Dosya ve Dizin İzinleri: 644 ve 755 Standartları** konusunda kurumsal bir WordPress web sitesine sahipseniz, yaşanan teknik aksaklıklar doğrudan müşteri kaybına, Google arama sıralamalarının gerilemesine ve şirketinizin dijital prestijinin zedelenmesine yol açabilir. Bu rehberimizde hatanın kök nedenlerini, sunucu ve kod seviyesindeki kesin çözüm adımlarını detaylı olarak inceliyoruz.

    ** Uzman Notu:** Web sitenizde herhangi bir yapılandırma, kod veya veritabanı müdahalesi yapmadan önce mutlaka sunucunuzdan tam zaman damgalı bir veritabanı ve dosya yedeği (backup) alınız.

### Sorunun Olası Kök Nedenleri

WordPress altyapısında **Dosya ve Dizin İzinleri** durumunun tetiklenmesine yol açan en yaygın 4 temel etken şunlardır:

    
* **Sunucu Kaynak ve Bellek Yetersizliği:** PHP hafıza sınırının (memory_limit) veya işlem zaman aşımının (max_execution_time) aşılması.
    
* **Eklenti ve Tema Çakışmaları:** Yeni güncellenen bir yazılımın mevcut çekirdek WordPress dosyaları veya diğer eklentilerle uyumsuzluk yaşaması.
    
* **Bozuk Yapılandırma Dosyaları:** `.htaccess`, `wp-config.php` veya sunucu Nginx/Apache konfigürasyonundaki hatalı direktifler.
    
* **Veritabanı İndeks ve Bağlantı Hataları:** MySQL servisinin aşırı yük altında kilitlenmesi veya tabloların çökmesi.

### 1. Adım: Hata Ayıklama (Debug) Modu ile Sorunun Kaynağını Belirleyin

Karanlıkta çözüm aramak yerine sorunun hangi eklentiden veya kod satırından kaynaklandığını nokta atışı tespit etmek gerekir. Bunun için web sitenizin ana dizininde yer alan `wp-config.php` dosyasını açarak aşağıdaki satırları ekleyin:

```php
// Hata ayıklama modunu sessiz günlük kaydıyla açın
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
@ini_set( 'display_errors', 0 );
```

Bu ayar ziyaretçilerinize herhangi bir hata kodu göstermez; ancak sunucunuzun arka planında `/wp-content/debug.log` dosyasını oluşturarak çöken fonksiyonu satır numarasıyla birlikte kaydeder.

### 2. Adım: Sunucu ve PHP Parametrelerini Optimize Edin

Kurumsal altyapılarda modern WordPress eklentileri (WooCommerce, Elementor, WPML vb.) yüksek bellek ve yürütme süresi talep eder. Hosting kontrol panelinizden veya `.user.ini` dosyanızdan şu limitleri güncelleyin:

```ini
memory_limit = 256M
max_execution_time = 300
upload_max_filesize = 64M
post_max_size = 64M
max_input_vars = 3000
```

Eğer paylaşımlı sunucuda bu değerler güncellenemiyorsa, `wp-config.php` dosyasına şu satırı ekleyerek WordPress'in bellek sınırını 256MB'a yükseltebilirsiniz:

```php
define( 'WP_MEMORY_LIMIT', '256M' );
define( 'WP_MAX_MEMORY_LIMIT', '512M' );
```

### 3. Adım: Yapılandırma Dosyalarını ve Kalıcı Bağlantıları Sıfırlayın

Bozuk yönlendirme kuralları ve güvenlik eklentilerinin bıraktığı hatalı kurallar erişimi engelleyebilir. Standart temiz `.htaccess` kurallarını uygulayınız:

```apache
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
```

Ardından WordPress yönetim panelinden **Ayarlar → Kalıcı Bağlantılar** sekmesini ziyaret edip herhangi bir değişiklik yapmadan **Değişiklikleri Kaydet** butonuna basarak Apache yönlendirme önbelleğini tazeleyin.

### 4. Adım: Veritabanı ve Önbellek Kalıntılarını Temizleyin

Süresi dolmuş transient kalıntıları ve silinmiş eklentilerin bıraktığı sorgular veritabanını yavaşlatır. phpMyAdmin veya WP-CLI üzerinden tabloları optimize edebilirsiniz:

```bash
# WP-CLI ile önbellek ve geçici verileri temizleyin
wp cache flush
wp transient delete --expired
wp db optimize
```

    ** Kritik Güvenlik Uyarısı:** FTP veya sunucu erişim bilgilerinizi güvenli olmayan kanallardan kimseyle paylaşmayınız. WordPress Uzmanı olarak tüm şifrelerinizi AES-256-GCM ile şifrelenmiş kasamızda tutuyor, iş teslim edildiğinde otomatik olarak sistemden kalıcı olarak siliyoruz.

### Ne Zaman Uzmana Başvurmalı?

Eğer yukarıdaki adımlara rağmen sitenizdeki **Dosya ve Dizin İzinleri** sorunu giderilemiyorsa, veritabanı tablolarında derin bozulmalar, çekirdek dosyalara bulaşmış zararlı kodlar (malware) veya sunucu seviyesinde modül çakışmaları mevcut olabilir. Bu tür kritik durumlarda deneme yanılma yapmak veri kaybına yol açabileceğinden, kurumsal sözleşmeli ve faturalı destek masamızdan **Yazılı Teklif** talep ederek uzman müdahalesi almanız önerilir.

---

> 📌 **Orijinal Kaynak:** Bu makale ilk olarak [WordPress Uzmanı](https://wordpressuzmani.com/blog/wordpress-dosya-ve-dizin-izinleri-chmod-standartlari) üzerinde yayınlanmıştır.
>
> 🛠️ **WordPress Teknik Destek & Danışmanlık:** Hız optimizasyonu, güvenlik temizliği, sunucu yönetimi ve özel geliştirme ihtiyaçlarınız için [WordPress Uzmanı](https://wordpressuzmani.com) adresinden bize ulaşabilirsiniz.
