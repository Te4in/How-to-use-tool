# Gobuster Guide

## What is Gobuster?
Gobuster is a directory, DNS, and virtual host brute-force tool.

Official project:
https://github.com/OJ/gobuster

---

# Basic Syntax

```bash
gobuster MODE [options]
```

---

# Directory Enumeration

## Basic directory scan

```bash
gobuster dir -u http://example.com \
-w /usr/share/wordlists/dirb/common.txt
```

---

## Specify extensions

```bash
gobuster dir -u http://example.com \
-w wordlist.txt -x php,txt,html
```

---

## Thread count

```bash
gobuster dir -u http://example.com \
-w wordlist.txt -t 50
```

---

# DNS Enumeration

```bash
gobuster dns -d example.com \
-w subdomains.txt
```

---

# Virtual Host Enumeration

```bash
gobuster vhost -u http://example.com \
-w subdomains.txt
```

---

# Fuzz Mode

```bash
gobuster fuzz -u http://example.com/FUZZ \
-w wordlist.txt
```

---

# Useful Flags

| Flag | Description |
|---|---|
| -u | URL |
| -w | Wordlist |
| -t | Threads |
| -x | Extensions |
| -k | Skip TLS verify |
| -o | Output file |

---

# Common Wordlists

```bash
/usr/share/wordlists/dirb/common.txt
/usr/share/seclists/
```

---

# Notes
- Large wordlists can create heavy traffic.
- Respect rate limits and authorization boundaries.
