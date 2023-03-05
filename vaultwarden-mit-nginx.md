 # Vaultwarden mit Nginx    
Vaultwarden Docker Container mit lokalem HTTPS durch Nginx Reverse Proxy   
 --- 
 ## Dokumentation:   
- [https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-18-04)    
- [https://github.com/dani-garcia/vaultwarden](https://github.com/dani-garcia/vaultwarden)    
   
   
 ## Voraussetzung:   
- Openssl auf dem Server   
- Docker auf dem Server   
   
 ## Ablauf:   
Dateistruktur:   
```
vaultwarden
  | -> nginx
  |  |--> config
  |  |  |-- nginx.conf
  |  |  |-- > snippets
  |  |      |--> self-signed.conf
  |  |      |--> ssl-params.conf
  |  |--> ssl
  |     |--> certs
  |     |--> private

```
im Ordner *nginx/ssl,* werden die SSL-Zertifikate generiert:`
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout private/private.key -out certs/cert.crt`   
→ dabei enstehen die Datein cert.crt im Ordner ssl/certs (Public Key) und private.key im Ordner ssl/private (geheimer Privater Key)   
   
Im Ordner *nginx/config *eine Diffie-Hellman Gruppe generieren:`
sudo openssl dhparam -out dhparam.pem 4096`   
→ dies kann einige Zeit dauern   
   
Die Datei *self-signed.conf*:   
```
ssl_certificate /etc/ssl/certs/cert.crt;
ssl_certificate_key /etc/ssl//private/private.key;
```
   
Die Datei *ssl-params.conf*:   
```
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/nginx/dhparam.pem; 
ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
ssl_ecdh_curve secp384r1;
ssl_session_timeout  10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```
   
Die Datei *nginx.conf*:   
```
http{
    server {
        listen 443 ssl;
        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;

        access_log            /var/log/nginx/test.access.log;

        location / {
          proxy_set_header        Host $host;
          proxy_set_header        X-Real-IP $remote_addr;
          proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header        X-Forwarded-Proto $scheme;

          proxy_pass          http://vault;
          proxy_read_timeout  90;
        }
    }
}

events {
    worker_connections 1024;
}

```
   
Docker Compose Config, wie folgt:   
```
version: "1.0"
services:
    vw:
      image: vaultwarden/server:latest
      container_name: vaultwarden
      ports:
        - 3012:3012
      volumes:
        - /PATH_TO/vaultwarden:/data/
      restart: "always"
    nginx:
      image: nginx:latest
      container_name: vw_nginx
      ports:
        - PORT:443
      links:
        - vw:vault
      volumes:
        - /PATH_TO/vaultwarden/nginx/config:/etc/nginx
        - /PATH_TO/vaultwarden/nginx/ssl:/etc/ssl
      restart: unless-stopped

```
**PATH\_TO**und **PORT** zu den eigenen Werten ändern   
nun kann der Docker Container gestartet werden und wenn alles funktioniert hat, ist der Vaultwarden Server jetzt unter dem vorgegebenen Port erreichbar   
   
