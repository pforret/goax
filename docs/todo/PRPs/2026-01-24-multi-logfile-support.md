# PRP: Multi-Log File Support with Gzipped Files

## Summary

Enable `goax.sh run` to process multiple nginx log files including rotated and gzipped archives (e.g., `access.log`, `access.log.1`, `access.log.2.gz`).

## Context

### Current Behavior

The `do_run()` function in `goax.sh:275-337` currently supports glob patterns via the `-l` option:

```bash
local log_file="${LOG_FILE:-${ACCESS_LOG:-/var/log/nginx/access.log}}"
# shellcheck disable=SC2086
for f in $log_file; do
  [[ -f "$f" ]] && log_files+=("$f")
done
```

However, it uses `cat` to combine files (line 310):
```bash
cat "${log_files[@]}" > "$tmp_all"
```

This fails for `.gz` files which require decompression.

### Target Behavior

Support command like:
```bash
./goax.sh run -l "/var/log/nginx/toolstud.io-access.log /var/log/nginx/toolstud.io-access.log.1 /var/log/nginx/toolstud.io-access.log.2.gz"
```

Or via glob:
```bash
./goax.sh run -l "/var/log/nginx/toolstud.io-access.log*"
```

## Technical Approach

### GoAccess Native Support

From [GoAccess Manual](https://goaccess.io/man):

```bash
# Multiple uncompressed files
goaccess access.log access.log.1

# Compressed files via zcat + stdin
zcat access.log.*.gz | goaccess access.log access.log.1 -
```

The `-` tells goaccess to read from stdin, allowing mixing piped compressed data with regular files.

### Implementation Strategy

Modify `do_run()` to:
1. Separate log files into two arrays: `plain_files[]` and `gz_files[]`
2. Use conditional logic to handle three scenarios:
   - Only plain files: `cat plain_files > tmp`
   - Only gz files: `zcat gz_files > tmp`
   - Mixed: `{ cat plain_files; zcat gz_files; } > tmp`

### macOS Compatibility

macOS uses `gunzip -c` instead of `zcat`. Add detection:
```bash
local zcat_cmd="zcat"
[[ "$os_kernel" == "Darwin" ]] && zcat_cmd="gunzip -c"
```

## Implementation Tasks

1. **Add gzip file detection** in `do_run()` (~line 283-290)
   - Split `log_files` array into `plain_files` and `gz_files`
   - Check each file extension for `.gz`

2. **Add macOS-compatible decompression** (~line 307)
   - Detect OS and set appropriate command
   - Use `gunzip -c` for Darwin, `zcat` for Linux

3. **Update temp file creation** (~line 307-310)
   - Replace simple `cat` with conditional logic
   - Handle plain-only, gz-only, and mixed scenarios

4. **Update output messaging** (~line 297)
   - Show which files are plain vs compressed
   - Display total count

## Code Changes

### File: `goax.sh`

**Location: `do_run()` function (lines 275-337)**

Replace lines 283-310 with:

```bash
function do_run() {
  IO:log "run"
  Os:require "goaccess"

  local log_file="${LOG_FILE:-${ACCESS_LOG:-/var/log/nginx/access.log}}"
  local output_dir="${OUTPUT:-${OUTPUT_DIR:-/var/www/stats}}"
  local log_format="${FORMAT:-${LOG_FORMAT:-COMBINED}}"

  # Separate plain and gzipped files
  local plain_files=()
  local gz_files=()
  local f
  # shellcheck disable=SC2086
  for f in $log_file; do
    if [[ -f "$f" ]]; then
      if [[ "$f" == *.gz ]]; then
        gz_files+=("$f")
      else
        plain_files+=("$f")
      fi
    fi
  done

  local total_files=$(( ${#plain_files[@]} + ${#gz_files[@]} ))
  if [[ $total_files -eq 0 ]]; then
    IO:die "No log files found matching: $log_file"
  fi

  # Create output directory if needed
  [[ ! -d "$output_dir" ]] && mkdir -p "$output_dir"

  IO:print "Log file(s): $total_files total (${#plain_files[@]} plain, ${#gz_files[@]} gzipped)"
  IO:print "Output dir: $output_dir"
  IO:print "Format: $log_format"

  # Set decompression command (macOS vs Linux)
  local zcat_cmd="zcat"
  [[ "$os_kernel" == "Darwin" ]] && zcat_cmd="gunzip -c"

  # Bot detection patterns (unchanged)
  local bot_pattern="bot|crawler|spider|slurp|bingpreview|googlebot|yandex|baidu|semrush|ahref|mj12|dotbot|petalbot|bytespider|gptbot|claudebot|facebookexternalhit|twitterbot|linkedinbot|whatsapp|telegram|applebot|duckduck"
  local llm_pattern="gptbot|chatgpt-user|claudebot|claude-web|anthropic|bytespider|ccbot|perplexitybot|cohere-ai|meta-externalagent|amazonbot|youbot|ai2bot|diffbot|omgili|iaskspider"

  # Create temp file with all logs combined
  local tmp_all
  tmp_all=$(Os:tempfile log)

  # Combine all log files (handling both plain and gzipped)
  if [[ ${#gz_files[@]} -eq 0 ]]; then
    # Only plain files
    cat "${plain_files[@]}" > "$tmp_all"
  elif [[ ${#plain_files[@]} -eq 0 ]]; then
    # Only gzipped files
    $zcat_cmd "${gz_files[@]}" > "$tmp_all"
  else
    # Mixed: plain + gzipped
    { cat "${plain_files[@]}"; $zcat_cmd "${gz_files[@]}"; } > "$tmp_all"
  fi

  # Generate reports (unchanged from here)
  # ... rest of function unchanged ...
```

## Validation

### Test Commands

```bash
# Test with single plain file
./goax.sh run -l /var/log/nginx/access.log -o /tmp/stats

# Test with multiple plain files
./goax.sh run -l "/var/log/nginx/access.log /var/log/nginx/access.log.1" -o /tmp/stats

# Test with gzipped file only
./goax.sh run -l "/var/log/nginx/access.log.2.gz" -o /tmp/stats

# Test with mixed files (the target use case)
./goax.sh run -l "/var/log/nginx/toolstud.io-access.log /var/log/nginx/toolstud.io-access.log.1 /var/log/nginx/toolstud.io-access.log.2.gz" -o /tmp/stats

# Test with glob pattern
./goax.sh run -l "/var/log/nginx/toolstud.io-access.log*" -o /tmp/stats
```

### Expected Output

```
Log file(s): 3 total (2 plain, 1 gzipped)
Output dir: /tmp/stats
Format: COMBINED
... Generating all.html...
... Generating bots.html...
... Generating nobots.html...
... Generating llmbots.html...
✅  Reports generated in /tmp/stats
```

### Verification Steps

1. Check that all HTML reports are created
2. Verify line counts in combined temp file match sum of individual files
3. Test on both macOS and Linux if possible

## References

- [GoAccess Manual - Multiple Log Files](https://goaccess.io/man)
- [GoAccess GitHub Issue #2333 - gz logs](https://github.com/allinurl/goaccess/issues/2333)
- [David Yin's Blog - Multiple Nginx Log Files](https://www.yinfor.com/2022/02/use-goaccess-1-5-5-to-generate-report-with-multiple-nginx-log-files.html)

## Unresolved Questions

None - implementation path is clear.

---

**Confidence Score: 9/10**

High confidence because:
- Simple file extension check for `.gz` detection
- Well-documented goaccess behavior for stdin piping
- Existing bashew patterns for OS detection (`$os_kernel`)
- Minimal changes to existing code structure
- Clear test cases available
