---
name: logs-explorer
description: Use when analyzing log files, debugging errors, searching through logs, or investigating issues from log data. Triggers on "analyze logs", "search logs", "debug error", "log file", "parse logs", "log patterns", "investigate crash".
---

# Logs Explorer

Systematic log analysis for finding patterns, debugging issues, and extracting insights.

## When to Use This Skill

- Debugging errors or crashes from log files
- Searching for patterns across large log files
- Investigating security incidents
- Finding performance bottlenecks
- Extracting metrics from logs
- Comparing logs before/after a change

## Log File Locations

Common locations on the system:
```bash
# OpenClaw logs
~/.openclaw/logs/
~/.openclaw/logs/gateway.log

# System logs (Linux)
/var/log/syslog
/var/log/messages
/journal/log

# Application logs (check docs or config)
# Usually in ~/logs, /var/log/<app>, or <project>/logs
```

## Core Commands

### Search Patterns

```bash
# Find lines containing error
grep -i "error" /path/to/log

# Find with context (3 lines before and after)
grep -B3 -A3 "ERROR" /path/to/log

# Case-insensitive search
grep -i "exception" /path/to/log

# Multiple patterns (OR)
grep -E "error|warning|fatal" /path/to/log

# Multiple patterns (AND)
grep "error" /path/to/log | grep "database"

# Find lines NOT containing pattern
grep -v "DEBUG" /path/to/log
```

### Analyze by Time Range

```bash
# Logs from last hour
grep "$(date -d '1 hour ago' '+%Y-%m-%d %H:%M')" /path/to/log

# Between two timestamps
sed -n '/2026-04-01 10:00/,/2026-04-01 11:00/p' /path/to/log

# Recent errors only
tail -1000 /path/to/log | grep -i error
```

### Extract and Format

```bash
# Extract timestamps
grep "ERROR" /path/to/log | awk '{print $1, $2}'

# Count by type
grep -o "ERROR\|WARN\|INFO" /path/to/log | sort | uniq -c

# Get unique error messages
grep "ERROR" /path/to/log | cut -d: -f4- | sort | uniq

# Extract stack traces
awk '/^  at / {stack = stack "\n" $0} /^[^ ]/ && stack {print stack; stack=""}' /path/to/log
```

## Log Level Patterns

| Level | Pattern | Severity |
|-------|---------|----------|
| ERROR | `ERROR`, `FATAL`, `CRITICAL` | High - immediate action needed |
| WARN | `WARN`, `WARNING` | Medium - investigate soon |
| INFO | `INFO`, `NOTICE` | Low - informational |
| DEBUG | `DEBUG`, `TRACE`, `VERBOSE` | Debug only |

## Common Log Formats

### JSON Logs (structured)
```bash
# Extract field from JSON logs
grep "ERROR" app.log | jq -r '.message'

# Find errors by user
grep "ERROR" app.log | jq -r 'select(.user_id == "123") | .message'
```

### Apache/Nginx Access Logs
```bash
# Find 5xx errors
grep " 5[0-9][0-9] " access.log

# Most requested paths
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -20

# Top IPs by request count
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20
```

## Analysis Patterns

### Error Frequency Over Time
```bash
# Errors per hour
grep "ERROR" /path/to/log | awk '{print $2}' | cut -d: -f1 | sort | uniq -c
```

### Find Related Errors
```bash
# Errors around the same time (within 1 minute)
grep "2026-04-12 15:4[0-9]" /path/to/log | grep ERROR
```

### Memory/Performance Issues
```bash
# Out of memory
grep -i "out of memory\|oom\|killed" /path/to/log

# High CPU
grep -i "cpu\|load\|timeout" /path/to/log
```

## Quick Reference

```bash
# Watch logs in real-time
tail -f /path/to/log

# Watch only errors
tail -f /path/to/log | grep -i error

# Count lines in large log
wc -l /path/to/log

# View last 100 lines with line numbers
tail -n 100 /path/to/log | cat -n

# Find log files by size
find . -name "*.log" -ls -h | sort -h -k5 -r | head -10
```

## Anti-Patterns

- Don't grep for everything at once - narrow down first
- Don't ignore WARN messages - they often precede ERRORS
- Don't delete old logs until you've analyzed them
- Don't assume the first error is the root cause