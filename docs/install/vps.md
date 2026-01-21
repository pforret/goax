# Installing goax on a VPS

Generic guide for Linux VPS (Ubuntu, Debian, CentOS, etc.).

## 1. Install goax

```bash
cd /opt
sudo git clone https://github.com/pforret/goax.git
sudo chmod +x /opt/goax/goax.sh
sudo ln -s /opt/goax/goax.sh /usr/local/bin/goax
```

## 2. Install goaccess

```bash
goax install
```

Or manually per distro:

| Distro | Command |
|--------|---------|
| Ubuntu/Debian | `sudo apt install goaccess` |
| CentOS/RHEL | `sudo yum install goaccess` |
| Fedora | `sudo dnf install goaccess` |
| Arch | `sudo pacman -S goaccess` |
| Alpine | `sudo apk add goaccess` |

## 3. Configure

```bash
cd /opt/goax
goax config
```

Common log locations:

| Web server | Log path |
|------------|----------|
| Nginx (Debian) | `/var/log/nginx/access.log` |
| Nginx (CentOS) | `/var/log/nginx/access.log` |
| Apache (Debian) | `/var/log/apache2/access.log` |
| Apache (CentOS) | `/var/log/httpd/access_log` |

## 4. Setup protected folder

```bash
sudo goax folder statsadmin
```

### Nginx config

Add to your server block:
```nginx
location /stats {
    auth_basic "Statistics";
    auth_basic_user_file /etc/nginx/.htpasswd;
    alias /var/www/stats;
    index report.html;
}
```

Reload: `sudo systemctl reload nginx`

### Apache config

Create `/var/www/stats/.htaccess`:
```apache
AuthType Basic
AuthName "Statistics"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

Enable mod_auth: `sudo a2enmod auth_basic`

Reload: `sudo systemctl reload apache2`

## 5. Generate report

```bash
sudo goax run
```

## 6. Setup cron

```bash
sudo crontab -e
```

```cron
# Hourly
0 * * * * /opt/goax/goax.sh run

# Daily at 3am
0 3 * * * /opt/goax/goax.sh run

# Every 15 minutes
*/15 * * * * /opt/goax/goax.sh run
```

## Permissions

If running as non-root user, add to `adm` group:
```bash
sudo usermod -aG adm $USER
```

Or adjust log file permissions:
```bash
sudo chmod 644 /var/log/nginx/access.log
```
