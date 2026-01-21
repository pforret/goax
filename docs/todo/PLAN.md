# goax Implementation Plan

## 1. `goax.sh install` - Install goaccess ✅ DONE

Install goaccess binary with OS detection.

- [x] if `which goaccess` finds a binary, show path and version
- [x] if not, use `Os:require` to show install instructions per OS

---

## 2. `goax.sh config` - Create config file ✅ DONE

Interactive setup for `goax.env`:

- [x] Detect common log locations
- [x] Ask for log file path
- [x] Ask for output directory
- [x] Ask for log format
- [x] Save to `goax.env`

Pending:
- [ ] Multi-setup: separate reports per website
- [ ] Create `/etc/goaccess/goaccess.conf` file

---

## 3. `goax.sh folder` - Setup protected stats folder ✅ DONE

- [x] Detect nginx or apache
- [x] Create htpasswd file with user
- [x] Show config snippet for nginx
- [x] Show config snippet for apache

---

## 4. `goax.sh run` - Generate reports ✅ DONE

- [x] Support single log file
- [x] Support glob patterns (multiple files)
- [x] Generate `all.html` (all traffic)
- [x] Generate `bots.html` (bots only)
- [x] Generate `nobots.html` (no bots)
- [x] Generate `index.html` (navigation wrapper)
- [x] Bot detection via user-agent patterns

---

## Option:config ✅ DONE

```bash
choice|1|action|action to perform|install,config,folder,run,check,env,update
option|l|LOG_FILE|nginx access log path|
option|o|OUTPUT|output folder for reports|
option|F|FORMAT|log format (COMBINED/COMMON/...)|COMBINED
```

---

## Documentation ✅ DONE

- [x] docs/index.md - main documentation
- [x] docs/install/ubuntu.md
- [x] docs/install/digitalocean.md
- [x] docs/install/vps.md

---

## Future Enhancements

1. ~~Support multiple log files~~ ✅ Done via glob patterns
2. Include GeoIP database setup in `install`?
3. ~~Bot filtering~~ ✅ Done - separate reports generated
4. Real-time HTML mode: `goax.sh live`?
5. JSON/CSV output formats?
6. Per-website multi-config support?
