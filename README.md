# Django için genel bilgiler.

## Django Projesi için Apache Konfigürasyonu

Apache, gelen istekleri yönetmek için konfigürasyon dosyalarını kullanır. Her bir site veya hizmet için `<VirtualHost>` etiketleri içerisinde kurallar tanımlanır. Bu kurallar, belirli bir domain (alan adı) ile gelen isteklerin nasıl yönlendirileceğini, hangi klasöre ulaşacağını ve hangi protokollerle çalışacağını belirler. 

Konfigürasyon dosyaları `/etc/apache2/sites-available/` dizininde bulunur. Bu rehberde, hem HTTP (80 portu) hem de HTTPS (443 portu) için ayrı dosyalar oluşturacağız.


### HTTP Konfigürasyonu (80 Portu)

HTTP konfigürasyonu, güvenli olmayan (non-secure) istekleri ele alır. Bu yapılandırma genellikle gelen tüm HTTP isteklerini HTTPS'e yönlendirmek için kullanılır.

Örnek (`dashboard.conf`):
```apache
<VirtualHost *:80>
    ServerName dashboard.omnichannel.sirket.im
    ServerAlias dashboard.omnichannel.sirket.im
    ServerAdmin webmaster@localhost
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    RewriteEngine On
    # www ile gelen istekleri yönlendirme
    RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
    RewriteRule ^(.*)$ https://%1/$1 [R=301,L]
    # HTTP isteklerini HTTPS'e yönlendirme
    RewriteCond %{SERVER_PORT} !^443$
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>
```
- Burada VirtualHost  tag'i içerisinde virtualhost oluşturuyorsun ayağa kalkacak her site için.
- ServerName ve Server Alias domain geliyor. O domainle bu servera erişenleri bu dosyadaki kurallara göre karşılıyor server
- ServerName ve ServerAlias: Bu konfigürasyon dosyasının yöneteceği domain(ler).
ErrorLog ve CustomLog: Hata ve erişim loglarının nereye yazılacağını belirler.

RewriteEngine: URL yönlendirme mekanizmasını etkinleştirir.

```apache
RewriteEngine On
RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
        RewriteRule ^(.*)$ https://%1/$1 [R=301,L]
        RewriteCond %{SERVER_PORT} !^443$
        RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
```
Şu bölüm gelen sorguları https'e yönlendiriyor.\
Bu 80 portundan gelenler için geçerli ayar dosyası. 80 portu standart http://... portu.

```apache
<VirtualHost *:443>
    ServerName dashboard.omnichannel.sirket.im
    ServerAlias dashboard.omnichannel.sirket.im
    ServerAdmin webmaster@localhost

#     ProxyPass / http://localhost:9000/
#     ProxyPassReverse / http://localhost:9000/

    # WebSocket Proxy Ayarları
#     ProxyPassMatch ^/(.*)$ ws://localhost:9000/$1

        DocumentRoot /var/www/omnichannel-dashboard/build/dashboard
        <Directory /var/www/omnichannel-dashboard/build/dashboard>
                Options Indexes FollowSymLinks
                AllowOverride All
                Require all granted

                # React uygulaması için Rewrite kuralları
                RewriteEngine On
                RewriteBase /
                RewriteCond %{REQUEST_FILENAME} !-f
                RewriteCond %{REQUEST_FILENAME} !-d
                RewriteRule ^ index.html [L]
        </Directory>

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/dashboard.omnichannel.sirket.im/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/dashboard.omnichannel.sirket.im/privkey.pem
</VirtualHost>
```
Bu da 443 portundan gelenler için. 443 de standart https:// portu.
DocumentRoot ana dizini belirliyor. Yani https://dashboard.omnichannel.sirket.im 'e girenler "/var/www/omnichannel-dashboard/build/dashboard" bu dizine gidiyorlar.

Directory tagı ile başlayan kısımda ilgili dizinin ayarları var. 
Options Indexes FollowSymLinks
AllowOverride All
Require all granted
bu bölüm o dizindeki tüm dosyaları görüntülenir açılır kılıyor.

RewriteEngine On
RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.html [L]
Bu bölüm gelen url'e karşılık bir dosya yoksa onları index.html'e yönlendiriyor

SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/dashboard.omnichannel.sirket.im/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/dashboard.omnichannel.sirket.im/privkey.pem
Burası da SSL sertifikasını verdiğimiz yer.

ProxyPreserveHost On
ProxyPass / http://localhost:9009/
ProxyPassReverse / http://localhost:9009/
Storefrontta dosyaları alamıyoruz onu 9009 portunda ayağa kaldırdım. Onun için de yukarıdaki ayarı verdim. Bu da gelen sorguları loalhost:9009'a yönlendiriyor.
