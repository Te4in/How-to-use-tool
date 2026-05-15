# Nikto Guide

## What is Nikto?
Nikto is a web server vulnerability scanner.

Official project:
https://github.com/sullo/nikto

---

# Basic Syntax

```bash
nikto -h TARGET
```

---

# Basic Examples

## Scan a website

```bash
nikto -h http://192.168.1.10
```

---

## Scan HTTPS

```bash
nikto -h https://example.com
```

---

## Specify port

```bash
nikto -h 192.168.1.10 -p 8080
```

---

## Save output

```bash
nikto -h http://example.com -o result.txt
```

---

## Output as HTML

```bash
nikto -h http://example.com -Format html -o report.html
```

---

## Use SSL

```bash
nikto -h example.com -ssl
```

---

# Useful Flags

| Flag | Description |
|---|---|
| -h | Host |
| -p | Port |
| -ssl | HTTPS |
| -o | Output file |
| -Format | Output format |
| -Tuning | Specific tests |

---

# Tuning Options

Example:

```bash
nikto -h http://example.com -Tuning 2
```

Common tuning categories:
- 1 = Interesting files
- 2 = Misconfiguration
- 3 = Information disclosure
- 4 = Injection

---

# Notes
- Nikto generates many requests.
- Best used during authorized vulnerability assessments.
