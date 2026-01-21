# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**goax** - Bash wrapper for [goaccess](https://goaccess.io/) to generate web stats from nginx access logs.

Target features:
- Overall stats generation
- Bot vs non-bot traffic separation
- goaccess installation helper
- Protected web folder setup (password-protected stats output)
- Cron job configuration for automated stat generation

## Architecture

Single-file bash script based on [pforret/bashew](https://github.com/pforret/bashew) template (v1.22.0).

### Key Sections in goax.sh

1. **Option:config()** (line ~17): Define CLI flags/options/parameters
2. **Script:main()** (line ~58): Main action dispatcher (case statement)
3. **Helper functions** (line ~112+): Implement actions like `do_action1`, `do_action2`

Everything below line 127 is bashew boilerplate - do not modify.

### Adding New Actions

1. Add choice to `Option:config()`: `choice|1|action|...|action1,action2,newaction,check,env,update`
2. Add case in `Script:main()`:
```bash
newaction)
  #TIP: use «$script_prefix newaction» to ...
  #TIP:> $script_prefix newaction
  do_newaction
  ;;
```
3. Implement `do_newaction()` function

### Common bashew patterns

- Require binary: `Os:require "goaccess"`
- User confirmation: `IO:confirm "Proceed?"`
- Debug output: `IO:debug "message"` (shown with -V flag)
- Normal output: `IO:print "message"`
- Error/exit: `IO:die "error message"`
- Temp file: `local tmp=$(Os:tempfile txt)`

## Commands

```bash
# Run script
./goax.sh <action> [options]

# Show help
./goax.sh -h

# Check environment
./goax.sh check

# Verbose mode
./goax.sh -V <action>
```

## Configuration

- Env files: `.env`, `.goax.env`, or `goax.env`
- Log dir: `$HOME/log/goax` (option -L)
- Temp dir: `/tmp/goax` (option -T)
