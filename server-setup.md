# Server-Setup Dokumentation
Strato VPS · Ubuntu 22.04 · nginx · Let's Encrypt

---

## Verzeichnisstruktur

```
/var/www/
└── 15jahrevoraus/
    └── html/          ← Git-Repo (stefmuell/15jahrevoraus-web)
        ├── .git/
        └── index.html

/etc/nginx/
├── sites-available/   ← Config-Dateien (eine pro Domain)
│   └── 15jahrevoraus.de
└── sites-enabled/     ← Symlinks auf aktive Configs
    └── 15jahrevoraus.de -> ../sites-available/15jahrevoraus.de

/etc/letsencrypt/live/
└── 15jahrevoraus.de/
    ├── fullchain.pem
    └── privkey.pem
```

---

## Neue Domain hinzufügen

### 1. Verzeichnis anlegen
```bash
mkdir -p /var/www/NEUEDOMAIN/html
chown -R www-data:www-data /var/www/NEUEDOMAIN
chmod -R 755 /var/www/NEUEDOMAIN
```

### 2. Repo klonen (falls GitHub-Repo vorhanden)
```bash
rmdir /var/www/NEUEDOMAIN/html
git clone https://github.com/stefmuell/REPO.git /var/www/NEUEDOMAIN/html
chown -R www-data:www-data /var/www/NEUEDOMAIN/html
```

### 3. nginx Virtual Host anlegen
```bash
nano /etc/nginx/sites-available/NEUEDOMAIN.de
```

Inhalt:
```nginx
server {
    listen 80;
    listen [::]:80;

    server_name NEUEDOMAIN.de www.NEUEDOMAIN.de;

    root /var/www/NEUEDOMAIN/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 4. Config aktivieren
```bash
ln -s /etc/nginx/sites-available/NEUEDOMAIN.de /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

### 5. SSL-Zertifikat
```bash
certbot --nginx -d NEUEDOMAIN.de -d www.NEUEDOMAIN.de
```

Certbot richtet HTTPS und HTTP→HTTPS Redirect automatisch ein.

---

## Updates deployen (bestehende Domain)

```bash
cd /var/www/15jahrevoraus/html && git pull
```

---

## Wichtige Befehle

| Aktion | Befehl |
|--------|--------|
| nginx Config testen | `nginx -t` |
| nginx neu laden | `systemctl reload nginx` |
| nginx Status | `systemctl status nginx` |
| SSL-Zertifikate anzeigen | `certbot certificates` |
| SSL-Renewal testen | `certbot renew --dry-run` |
| nginx Error Log | `tail -f /var/log/nginx/error.log` |
| nginx Access Log | `tail -f /var/log/nginx/access.log` |
| UFW Status | `ufw status` |

---

## Wichtige Pfade

| Was | Pfad |
|-----|------|
| Web-Root (pro Domain) | `/var/www/DOMAIN/html/` |
| nginx Configs | `/etc/nginx/sites-available/` |
| nginx aktive Sites | `/etc/nginx/sites-enabled/` |
| SSL-Zertifikate | `/etc/letsencrypt/live/` |
| nginx Error Log | `/var/log/nginx/error.log` |
| Certbot Log | `/var/log/letsencrypt/letsencrypt.log` |

---

## Auto-Renewal

Certbot erneuert Zertifikate automatisch via systemd-Timer.
Prüfen mit: `systemctl status certbot.timer`
