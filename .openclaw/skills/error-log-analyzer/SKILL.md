name: error-log-analyzer
description: Analyzes error logs and stack traces to identify root causes, patterns, and actionable insights. Use when asked to "analyze error logs", "debug a crash", "parse stack trace", "find error patterns", or when troubleshooting failures in applications, CI/CD pipelines, or system services. Supports common formats (Python, Node.js, Go, Java) and can correlate errors across multiple log sources.
---

# Error Log Analyzer

Parses, categorizes, and analyzes error logs and stack traces to identify root causes and patterns.

## When to Use This Skill

- Debugging application crashes or failures
- Analyzing CI/CD pipeline failures
- Troubleshooting system service errors
- Finding patterns across many error entries
- Generating actionable bug reports from raw logs
- On-call incident response

## Supported Formats

| Runtime | Stack Trace Format | Key Fields |
|---------|-------------------|------------|
| Python | `Traceback (most recent call last):` | File, line, function, exception type |
| Node.js | `at <function> (<file>:<line>:<col>)` | Stack frames, error message |
| Go | `goroutine N [state]:` + file:line | Goroutine, file, line, panic value |
| Java/Kotlin | `Caused by: <ExceptionType>: <msg>` | Exception class, message, suppressed |
| Ruby | `Traceback (skipping N lines):` | File, line, method |
| Systemd | `Failed with result <result>.` | Unit, result, exit code |

## Quick Analysis

```bash
# Basic error extraction
LOGFILE="logs/app.log"
grep -E "ERROR|FATAL|Exception|Traceback|PANIC|Caught:" "$LOGFILE" | head -50

# Find error frequency
grep -oE "\[ERROR\].*" "$LOGFILE" | sort | uniq -c | sort -rn | head -20

# Extract stack traces
awk '/Traceback|Error:|Exception:/{found=1} found{print} /^[a-zA-Z]/$/{found=0}' "$LOGFILE"

# Recent errors (last hour)
grep "$(date -d '1 hour ago' +%Y-%m-%dT%H)" "$LOGFILE" | grep -i error
```

## Python Traceback Parsing

```bash
python3 << 'EOF'
import re, sys

log = sys.stdin.read()

# Split into individual tracebacks
tracebacks = re.split(r'Traceback \(most recent call last\):', log)

for i, tb in enumerate(tracebacks[1:], 1):
    lines = tb.strip().split('\n')
    print(f"\n=== Traceback {i} ===")
    
    # Parse frames
    frames = []
    for line in lines:
        m = re.match(r'  File "(.+)", line (\d+), in (.+)', line)
        if m:
            frames.append(f"  📄 {m.group(1)}:{m.group(2)} in {m.group(3)}")
    
    # Last line = exception
    if lines:
        exc_line = lines[-1].strip()
        exc_type = re.match(r'(\w+Error[\w]*): (.+)', exc_line)
        if exc_type:
            print(f"❌ {exc_type.group(1)}: {exc_type.group(2)}")
    
    print("Call stack (innermost last):")
    for f in reversed(frames):
        print(f)
EOF
```

## Node.js Stack Trace Parsing

```bash
node << 'EOF'
const log = require('fs').readFileSync('/dev/stdin', 'utf8');

const stackLines = log.split('\n').filter(l => l.startsWith('    at '));

const frames = stackLines.map(line => {
  const m = line.match(/at (.+) \((.+):(\d+):(\d+)\)/);
  if (m) return { fn: m[1], file: m[2], line: m[3], col: m[4] };
  const m2 = line.match(/at (.+):(\d+):(\d+)/);
  if (m2) return { fn: '(anonymous)', file: m2[1], line: m2[2] };
  return { fn: line.trim() };
});

console.log(`\n❌ ${frames.length} stack frames found:\n`);
frames.slice(0, 10).forEach(f => {
  if (f.file) {
    console.log(`  📄 ${f.file}:${f.line}${f.fn !== '(anonymous)' ? ' in ' + f.fn : ''}`);
  } else {
    console.log(`  → ${f.fn}`);
  }
});
EOF
```

## Pattern Detection

```bash
# Categorize common errors
python3 << 'EOF'
import re, sys
from collections import Counter

log = sys.stdin.read()

categories = {
    "ConnectionError": r"(ConnectionRefused|ConnectionTimeout|HTTPConnectionPool|HY000)",
    "AuthError": r"(401|Unauthorized|AuthenticationError|PermissionDenied|AccessDenied)",
    "NotFound": r"(404|NotFound|ENOENT|No such file)",
    "Timeout": r"(Timeout|timed out|TIMEOUT|Killed - timeout)",
    "OOM/Kill": r"(OOM|Killed|out of memory|MemoryError)",
    "AssertionError": r"(AssertionError|assert|AssertionFailed)",
    "TypeError": r"TypeError",
    "ValueError": r"ValueError",
    "ImportError": r"(ImportError|ModuleNotFoundError|Cannot find module)",
    "SIGTERM/SIGKILL": r"(SIGTERM|SIGKILL|signal 9|signal 15)",
}

counts = Counter()
errors = []

for line in log.split('\n'):
    for cat, pattern in categories.items():
        if re.search(pattern, line, re.IGNORECASE):
            counts[cat] += 1
            errors.append((cat, line.strip()))
            break

print("## Error Category Summary\n")
for cat, count in counts.most_common():
    bar = '█' * min(count, 50)
    print(f"{cat:20s} {count:4d}  {bar}")

print("\n## Sample Errors by Category\n")
for cat, _ in counts.most_common(5):
    sample = next((e for e in errors if e[0] == cat), None)
    if sample:
        print(f"\n### {cat}")
        print(f"```\n{sample[1][:200]}\n```")
EOF
```

## Root Cause Analysis

```bash
# Common root cause patterns
python3 << 'EOF'
import re, sys

log = sys.stdin.read()

# Look for the "caused by" chain
causes = re.findall(r'(?:Caused by| Originated at): ([^\n]+)', log, re.IGNORECASE)
if causes:
    print("## Error Chain\n")
    for i, cause in enumerate(causes, 1):
        print(f"  {i}. {cause.strip()}")

# Find first occurrence in time-sorted logs
timestamps = re.findall(r'^\[?(\d{4}-\d{2}-\d{2}[T ]\d{2}:\d{2}:\d{2})', log, re.MULTILINE)
if timestamps:
    print(f"\n## Timeline\n")
    print(f"  First entry: {timestamps[0]}")
    print(f"  Last entry:  {timestamps[-1]}")
    print(f"  Total entries: {len(timestamps)}")

# Identify repeating patterns
lines = [l.strip() for l in log.split('\n') if 'ERROR' in l.upper() and len(l.strip()) > 20]
from collections import Counter
# Group by first 50 chars
groups = Counter(l[:80] for l in lines)
print(f"\n## Most Frequent Errors\n")
for pattern, count in groups.most_common(5):
    print(f"  [{count:3d}x] {pattern[:70]}...")
EOF
```

## Multi-File Log Correlation

```bash
# Correlate errors across multiple log files
python3 << 'EOF'
import re, sys
from pathlib import Path
from datetime import datetime

log_dir = Path("logs")
error_map = {}

for logfile in log_dir.glob("*.log"):
    content = logfile.read_text()
    # Extract timestamp + error tuples
    pattern = r'\[?(\d{4}-\d{2}-\d{2}[T ]\d{2}:\d{2}:\d{2})\]?\s*.*?(ERROR|Exception|Traceback|FATAL).*'
    for m in re.finditer(pattern, content, re.IGNORECASE):
        ts = datetime.fromisoformat(m.group(1).replace(' ', 'T'))
        key = f"{m.group(2).lower()}:{m.group(0)[80:150].strip()}"
        if key not in error_map:
            error_map[key] = []
        error_map[key].append((str(logfile.name), ts, m.group(0)[:120]))

print(f"## Correlated Errors Across {len(log_dir.glob('*.log'))} Files\n")
for err, occurrences in sorted(error_map.items(), key=lambda x: -len(x[1]))[:10]:
    print(f"### [{len(occurrences)}x] {err[:80]}")
    for source, ts, snippet in occurrences[:3]:
        print(f"  - {ts.strftime('%H:%M:%S')} in {source}: {snippet[:80]}")
    print()
EOF
```

## Output Format

```markdown
## Error Log Analysis Report

**Source:** `logs/app.log`
**Analyzed:** YYYY-MM-DD HH:MM
**Total lines:** 4,821 | **Errors:** 47 | **Unique errors:** 12

### Summary by Category

| Category | Count | % |
|----------|-------|---|
| ConnectionError | 18 | 38% |
| Timeout | 12 | 26% |
| TypeError | 8 | 17% |
| ImportError | 5 | 11% |
| OOM/Kill | 4 | 8% |

### Root Cause (Most Likely)

The primary error is **HTTPConnectionPool: connection refused** occurring 18 times
starting at 14:23:01. All errors originate from `src/api/client.py:87` during
the health check polling loop.

```
requests.exceptions.ConnectionError:
  HTTPConnectionPool(host='redis', port=6379): 
  Connection refused (化亨=111)
  ↳ src/api/client.py:87 in health_check()
  ↳ src/api/client.py:45 in start()
  ↳ main.py:12 in <module>
```

### Recommendation

1. **Immediate:** Check if Redis service is running (`systemctl status redis`)
2. **Check:** Network policy / firewall between app and Redis host
3. **Root cause:** App starts before Redis is ready — add `depends_on` in docker-compose
```

## Tools

- **saddock/log-parser**: Python log parsing utilities
- **node-log-parse**: Node.js log parser
- **lnav**: `brew install lnav` — Advanced log viewer with pattern detection
- **grok**: `pip install pygrok` — Regex pattern matching for logs
- **jq**: JSON log filtering and aggregation
- ** multitail**: `apt install multitail` — Multiple log files in one view

## Common Error Translations

| Error | Meaning | Common Fix |
|-------|---------|------------|
| `ConnectionRefused` | Service not listening | Start service, check port |
| `ENOENT` | File/directory not found | Check path, working directory |
| `PermissionDenied` | Access issue | chmod, chown, or user permissions |
| `ImportError` | Module not installed or path wrong | `pip install`, check PYTHONPATH |
| `HTTPErrors: 500` | Server-side crash | Check application logs |
| `SIGTERM` | Graceful shutdown requested | Usually normal shutdown |
| `SIGKILL` | Forced kill (OOM or manual) | Increase memory, check ulimits |
| `ETIMEDOUT` | Connection timeout | Firewall, network, or host down |
