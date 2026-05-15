# Kiterunner Guide

## What is Kiterunner?
Kiterunner is an API discovery and content enumeration tool.

Official project:
https://github.com/assetnote/kiterunner

---

# Basic Syntax

```bash
kr [command]
```

---

# Enumerate APIs

```bash
kr scan http://example.com
```

---

# Use a Wordlist

```bash
kr scan http://example.com \
-w routes-large.kite
```

---

# Specify Threads

```bash
kr scan http://example.com \
-w routes-large.kite -j 50
```

---

# Save Output

```bash
kr scan http://example.com \
-w routes-large.kite -o output.json
```

---

# Replay Requests

```bash
kr scan http://example.com \
-x 10
```

---

# Useful Flags

| Flag | Description |
|---|---|
| -w | Wordlist |
| -j | Threads |
| -o | Output file |
| -x | Replay count |

---

# Common Wordlists

Assetnote wordlists:
https://wordlists.assetnote.io/

---

# Notes
- Useful for API reconnaissance.
- Designed for authorized security testing only.
