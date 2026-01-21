# goax Implementation Plan

## 1. `goax.sh install` - Install goaccess

Install goaccess binary with OS detection.

* if `which goacccess` finds a binary, just show that and stop.
* if not, install via:
    - macOS: `brew install goaccess`
    - Debian/Ubuntu: `apt install goaccess`
    - RHEL/CentOS: `yum install goaccess`
    - Arch: `pacman -S goaccess`

---

## 2. `goax.sh config` - Create config file

Interactive setup for `.goax.env`:

* simple setup: 1 access log, 1 set of reports
* multi-setup: folder with multiple access logs, separate set of reports per website
```bash
ACCESS_LOG="/var/log/nginx/access.log"
#or ACCESS_LOG="/var/log/nginx/*access.log"
OUTPUT_DIR="/var/www/stats"
LOG_FORMAT="COMBINED"
GOACCESS_CONF=""  # optional custom goaccess.conf
```

- Detect common log locations: `/var/log/nginx/access.log`, `/var/log/apache2/access.log`
- Validate log file exists and is readable
- Create output directory if needed
- create `/etc/goaccess/goaccess.conf` file

---

## 3. `goax.sh folder` - Setup protected stats folder

Create password-protected web folder for stats output.

### Nginx
```bash
# Create htpasswd file
htpasswd -c /etc/nginx/.htpasswd statsuser

# Nginx config snippet
location /stats {
    auth_basic "Statistics";
    auth_basic_user_file /etc/nginx/.htpasswd;
    alias /var/www/stats;
}
```

### Apache
```bash
# .htaccess in stats folder
AuthType Basic
AuthName "Statistics"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

Dependencies: `Os:require "htpasswd" "apache2-utils"`

---

## 4. `goax.sh run` - Generate reports

Run goaccess to create HTML report from access logs.

```bash
do_run() {
  Os:require "goaccess"
  local log_file="${ACCESS_LOG:-/var/log/nginx/access.log}"
  local output="${OUTPUT_DIR:-/var/www/stats}/report.html"
  local log_format="${LOG_FORMAT:-COMBINED}"

  goaccess "$log_file" \
    --log-format="$log_format" \
    -o "$output"
}
```

Options:
- `-o html|json|csv` output format

Crontab example:
```
0 * * * * /path/to/goax.sh run
```

---

## Option:config changes

```bash
choice|1|action|action to perform|install,config,folder,run,check,env,update
option|l|LOG_FILE|nginx access log path|
option|o|OUTPUT|output folder for reports|
option|F|FORMAT|log format (COMBINED/COMMON/...)|COMBINED
```

---

## Unresolved Questions

1. Support multiple log files (e.g. `access.log*` with rotation)? -> optionally, yes
2. Include GeoIP database setup in `install`?
3. Bot filtering: separate action or option in `run`?
4. Real-time HTML mode: separate action `goax.sh live`?
