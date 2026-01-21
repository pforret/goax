# Installing goax on Ubuntu

Tested on Ubuntu 20.04, 22.04, 24.04.

## 1. Prerequisites

```bash
sudo apt update
sudo apt install -y git apache2-utils
```

## 2. Install goax

```bash
cd /opt
sudo git clone https://github.com/pforret/goax.git
sudo chmod +x /opt/goax/goax.sh
sudo ln -s /opt/goax/goax.sh /usr/local/bin/goax
```

## 3. Install goaccess

```bash
# Standard repo (older version)
sudo apt install -y goaccess

# Or latest version from official repo
echo "deb https://deb.goaccess.io/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/goaccess.list
wget -O - https://deb.goaccess.io/gnugpg.key | sudo apt-key add -
sudo apt update
sudo apt install -y goaccess
```

## 4. Configure

```bash
cd /opt/goax
goax config
```

Ubuntu defaults:

| Setting | Nginx | Apache |
|---------|-------|--------|
| ACCESS_LOG | `/var/log/nginx/access.log` | `/var/log/apache2/access.log` |
| OUTPUT_DIR | `/var/www/html/stats` | `/var/www/html/stats` |
| LOG_FORMAT | `COMBINED` | `COMBINED` |

## 5. Setup protected folder

```bash
sudo mkdir -p /var/www/html/stats
sudo goax folder statsadmin
```

### For Nginx

Edit `/etc/nginx/sites-available/default`:
```nginx
location /stats {
    auth_basic "Statistics";
    auth_basic_user_file /etc/nginx/.htpasswd;
    alias /var/www/html/stats;
    index report.html;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
```

### For Apache

```bash
sudo a2enmod auth_basic
sudo nano /var/www/html/stats/.htaccess
```

Add:
```apache
AuthType Basic
AuthName "Statistics"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

```bash
sudo systemctl reload apache2
```

## 6. Generate report

```bash
sudo goax run
```

## 7. Setup cron

```bash
sudo crontab -e
```

```cron
0 * * * * /opt/goax/goax.sh run >> /var/log/goax.log 2>&1
```

## 8. Logrotate (optional)

Create `/etc/logrotate.d/goax`:
```
/var/log/goax.log {
    weekly
    rotate 4
    compress
    missingok
    notifempty
}
```

## Firewall

If using UFW:
```bash
sudo ufw allow 'Nginx Full'
# or
sudo ufw allow 'Apache Full'
```
