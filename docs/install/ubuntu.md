# Installing goax on Ubuntu

Tested on Ubuntu 20.04, 22.04, 24.04. Assumes web server (Nginx/Apache) is already running.

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
goax config
```

Config file location:
- If script folder is writable: `/opt/goax/goax.env`
- Otherwise (no sudo): `~/.config/goax/goax.env`

Example for domain `example.com`:

| Setting    | Value                                   |
|------------|-----------------------------------------|
| ACCESS_LOG | `/var/log/nginx/example.com.access.log` |
| OUTPUT_DIR | `/var/www/example.com/stats`            |
| LOG_FORMAT | `COMBINED`                              |

## 5. Setup protected folder

```bash
sudo mkdir -p /var/www/example.com/stats
sudo goax folder statsadmin
```

### For Nginx

Edit your site config (e.g. `/etc/nginx/sites-available/example.com`):
```nginx
location /stats {
    auth_basic "Statistics";
    auth_basic_user_file /etc/nginx/.htpasswd;
    alias /var/www/example.com/stats;
    index report.html;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
```

### For Apache

```bash
sudo a2enmod auth_basic
```

Create `/var/www/example.com/stats/.htaccess`:
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

View at: `https://example.com/stats/`

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
