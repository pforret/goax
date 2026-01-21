# Installing goax on DigitalOcean

Guide for Ubuntu droplets with Nginx.

## 1. Install goax

```bash
# Clone the repo
cd /opt
sudo git clone https://github.com/pforret/goax.git
sudo chmod +x /opt/goax/goax.sh

# Add to PATH (optional)
sudo ln -s /opt/goax/goax.sh /usr/local/bin/goax
```

## 2. Install goaccess

```bash
goax install
# or manually:
sudo apt update && sudo apt install goaccess -y
```

## 3. Configure

```bash
cd /opt/goax
goax config
```

Typical DigitalOcean values:
```
ACCESS_LOG="/var/log/nginx/access.log"
OUTPUT_DIR="/var/www/html/stats"
LOG_FORMAT="COMBINED"
```

For multiple sites:
```
ACCESS_LOG="/var/log/nginx/*access.log"
```

## 4. Setup protected folder

```bash
sudo goax folder statsadmin
```

Add to `/etc/nginx/sites-available/default`:
```nginx
location /stats {
    auth_basic "Statistics";
    auth_basic_user_file /etc/nginx/.htpasswd;
    alias /var/www/html/stats;
    index report.html;
}
```

Reload nginx:
```bash
sudo nginx -t && sudo systemctl reload nginx
```

## 5. Generate first report

```bash
sudo goax run
```

View at: `https://your-droplet-ip/stats/`

## 6. Setup cron

```bash
sudo crontab -e
```

Add (hourly reports):
```cron
0 * * * * /opt/goax/goax.sh run >> /var/log/goax.log 2>&1
```

## Troubleshooting

**Permission denied on log files:**
```bash
sudo usermod -aG adm $USER
# logout/login required
```

**Nginx config test fails:**
```bash
sudo nginx -t
```

**Check goaccess version:**
```bash
goaccess --version
```
