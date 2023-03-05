# Mattermost auf Synology NAS installieren

Die Mattermost installation läuft auf zwei Seiten ab:
1. Auf Seite der Synology DSM 
2. Im Terminal des Synology NAs

## 1. Synology Seite:
Auf der Synology Seite erstellen wir einen Domainnamen mit HTTPS Zertifikat, eine Reverse Proxy Verbindung zum Mattermost Container.
### 1.Domainname erstellen:
Die Infos wurden nache einen Tutorial von Mariushosting übernommen, [Originalversion](https://mariushosting.com/synology-how-to-enable-https-on-dsm-7/).
- in Synology DSM  **Systemsteuerung** > **Netzwerk** > **Konnektivität** > "HTTP/2 Aktivieren" anschalten, danach mit Übernehmen bestätigen
- für die Verbindung mit dem Synology.me Service müssen wir den **Port 443** der DiskStation im Router freigeben.
- unter **Systemsteuerung** > **Externer Zugriff** > **DDNS** eine neue Vernindung hinzufügen
>Service: Synology\
>Hostname: [deinname].synology.me\
>Synology Account: hier mit deinem Account anmelden, bzw. neuen Account erstellen \
> Zertiikat von Let's Encrypt... **ankreuzen** \
> Heartbeat aktivieren **ankreuzen**
- nach dem bestätigen wird die DDNS Verbindung erstellt und ein HTTPS Zertifikat erstellt
- wenn alles klappt, steht in der Übericht unter Status, normal
- **Achtung an dieser Stelle gibt es neue Zertifikate auf dem Server, mögliche OPENVPN Verbindungen/Konfigurationen müssen mit dem neuem Zertifikat erneuert werden**
- unter **Systemsteuerung** > **Sicherheut** > **Zertifikate** findet man das neu erstellte HTTPS Zertifikat

### 2. Subdomain für Chat erstellen
Aus einem Tutorial von Mariushosting, [Originalversion](https://mariushosting.com/how-to-install-mattermost-on-your-synology-nas/).
- **Systemsteuerung** > **Anmeldeportal** > **Erweitert** > **Reverse Proxy** eine neue [Reverse Proxy](https://de.wikipedia.org/wiki/Reverse_Proxy) Verbindung erstellen
> Name: Anzeigename\
> **Quelle**:\
> Protokol: HTTPS \
> Host: {subdomain}.[deinname].synology.me \
> Port: 443 \
>  [x] HSTS aktivieren \
>  **Ziel** \
>  Protokol: HTTP \
>  Host: localhost \
>  Port: 80 (wird später angepasst, da es von der Mattermost Konfiguration abhägig ist)

- im zweiten Einstellungsgfeld [Custom Header] benutzerdefinierte Header für ein Websocket erstellen und die komplette Reverse Proxy verbindung speichern
- **Systemsteuerung** > **Sicherheit** > **Erweitert** [x] HTTP Kompression aktivieren

### 3. Docker installation von Mattermost vorbereiten
- **Systemsteuerung** > **Terminal&SNMP** > SSH Service auf Port 22 aktivieren
- Docker wenn nicht vorhanden direkt über das Synology Packet Center installieren
- in der File Station den evtl. neu erstellen Ordner "docker" finden
- darin neuen Ordner "mattermost" erstellen
- Pfad des Ordners anzeigen/kopieren

ab hier muss erfolgt das weitere vorgehen in Terminal.



## 2. SSH Terminal:
### 1. Docker Repository herunterladen
- Verbindung per SSH mit dem NAS herstellen (mit einem Admin Konto)
- per cd zu dem  in der FileStation erstelten Mattermost Ordner navigieren 
> _typischerweise_ `cd /volume1/docker/mattermost`
- auf der offizielen Mattermost Seite den ersten Kommand zum klonen des Git-Repository verwenden [Mattermost Seite](https://mattermost.com/deploy/)

> `git clone https://github.com/mattermost/docker && cd docker`\
> _es entsteht ein Ordner docker, in welchem alle weteren relevanten Datein leigen werden_
- eine Konfigurationsdatei aus der Beispielkonfiguration `example.env`erstellen (siehe Mattermost Doku Schritt 2)
- `cp env.example .env`
- diese Konfiguration muss nun bearbeitet werden

### 2. Mattermost Konfiguration
- gerade neu entstandene Datei `.env`kann entweder direkt im Synology Terminal bearbeitet werden, doch ist in diesem nur `vi`vorinstalliertm,  daher empfielt sich ein extermen Texteditor, entweder auf dem NAS oder per down und upload lokal auf einem PC/Laptop
- in der Konfiguraitonsdatei müssen zwei Textstellen zwingend geändert werden:
> DOMAIN=mm.example.com 

hier kommt die oben erstellte Url der Reverse Proxy hin
> DOMAIN={subdomain}.[deinname].synology.me

>MATTERMOST_IMAGE=mattermost-enterprise-edition

wird zu 

>MATTERMOST_IMAGE=mattermost-**team**-edition 

geändert. Die Version unter `MATTERMOST_IMAGE_TAG`kann nach belieben angepasst werden. Die neuste verfügbare Verion lässt sich unter in den Mattermost [Release Notes](https://docs.mattermost.com/install/self-managed-changelog.html) nachlesen.

- klug ist es auch die Zugangsdaten für die darunterliegende Postgresql Datenbank von den Standartwerten abzuändern
> POSTGRES_USER=mmuser
POSTGRES_PASSWORD=mmuser_password
POSTGRES_DB=mattermost

- unter `APP_PORT=8065`finden wir den Port, auf welchem der Mattermost Server später läuft, dieser muss später noch in der Reverse Proxy eingetragen werden

### 3. Ordnerstruktur anlegen
- der Befehl aus Schritt 3 der Mattermost Dokumentation im Orndern `mattermost/docker`ausführen
> dieser Befehl erstellt alle nötigen Ornder, welche später als Volumes in den Docker Container eingebunden werden

- ein weiterer Befehl erstellt benötigte Datenbank Ordner und gibt dem Mattermost Benutzer notwendige Rechte über diese Ordner

`mkdir -p volumes/db/var/lib/postgresql/data/ && sudo chown -R 2000:2000 ./volumes/`


- Terminalseitig kann jetzt der Container erstmals deployed werden. (in der Mattermost Anleitung Schritt 4)

`sudo docker-compose -f docker-compose.yml -f docker-compose.without-nginx.yml up -d`

## 3. Letzer Blick in DSM 
- unter **Systemsteuerung** > **Anmeldeportal** > **Erweitert** > **Reverse Proxy** die vorhin erstellte Reverse Proxy bearbeiten und unter **Ziel**
> Host: localhost \
> Port: 8065 (siehe Konfiguration APP_PORT)

- Reverse Proxy speichern und schließen
- jetzt sieht man in der Docker App bereits die neu erstellten Container und kann im Protokol des mattermost Containers den Einrichtungsfortschritt beobachten
- sobald keine neuen Einträge mehr erscheinen ist Mattermost unter der vorher festgelegten Domain errichbar
`https://{subdomain}.[deinname].synology.me` 

Viel Spaß ^^
