# Web Security Scanning Toolkit: Beginner Guide

A practical command reference for learning web reconnaissance and
security scanning with tools such as Nmap, WhatWeb, Nikto, Gobuster,
Dirb, FFUF, SSLScan, testssl.sh, and Nuclei.

> **Authorization and safety:** Run these tools only against systems you
> own or have explicit written permission to test. The examples below
> use `127.0.0.1` and a local practice application. Do not copy a target
> from the internet and scan it without permission. Directory discovery
> and vulnerability templates can generate significant traffic; use
> conservative settings and follow the system owner's scope and rate
> limits.

## Contents

1.  [What is web scanning?](#1-what-is-web-scanning)
2.  [Prepare a local practice
    target](#2-prepare-a-local-practice-target)
3.  [Install the tools on Kali
    Linux](#3-install-the-tools-on-kali-linux)
4.  [Understand a URL and web
    target](#4-understand-a-url-and-web-target)
5.  [Nmap](#5-nmap)
6.  [WhatWeb](#6-whatweb)
7.  [Nikto](#7-nikto)
8.  [Gobuster](#8-gobuster)
9.  [Dirb](#9-dirb)
10. [FFUF](#10-ffuf)
11. [Wafw00f](#11-wafw00f)
12. [SSLScan and testssl.sh](#12-sslscan-and-testsslsh)
13. [Nuclei](#13-nuclei)
14. [A safe scanning workflow](#14-a-safe-scanning-workflow)
15. [How to understand results](#15-how-to-understand-results)
16. [Common problems](#16-common-problems)
17. [Practice exercises](#17-practice-exercises)
18. [Quick command reference](#18-quick-command-reference)
19. [Official resources](#19-official-resources)

------------------------------------------------------------------------

## 1. What is web scanning?

Web security scanning is the process of examining a website or web
application to understand its exposed services, technologies,
configuration, and possible security weaknesses.

Scanning can help answer questions such as:

-   Is the web server reachable?
-   Which ports are open?
-   What web server or framework is exposed?
-   Are there publicly accessible paths that should not be public?
-   Is HTTPS configured correctly?
-   Are there known security issues that need manual verification?

A scanner does not prove that a website is secure. Tools can report
false positives and miss vulnerabilities. Always verify findings
carefully.

### Common categories of tools

  -----------------------------------------------------------------------
  Category                Example tools           Main purpose
  ----------------------- ----------------------- -----------------------
  Network and port        Nmap                    Discover hosts, ports,
  discovery                                       and services

  Technology              WhatWeb                 Identify web
  identification                                  technologies

  Web server checks       Nikto                   Check for known
                                                  server/configuration
                                                  issues

  Content/path discovery  Gobuster, Dirb, FFUF    Find paths using a
                                                  wordlist

  WAF identification      Wafw00f                 Identify possible web
                                                  application firewalls

  TLS checks              SSLScan, testssl.sh     Review HTTPS/TLS
                                                  configuration

  Template-based checks   Nuclei                  Check for known issues
                                                  using templates
  -----------------------------------------------------------------------

## 2. Prepare a local practice target

The safest way to learn is to scan a web server running on your own
computer.

Open a normal terminal:

``` bash
mkdir -p ~/web-scan-lab/site
cd ~/web-scan-lab/site
```

Create a small practice website:

``` bash
cat > index.html <<'EOF'
<!doctype html>
<html>
<head>
  <title>My Local Security Lab</title>
</head>
<body>
  <h1>Local Web Scanning Practice</h1>
  <p>This website is only for learning.</p>
</body>
</html>
EOF
```

Create a harmless discoverable directory and file:

``` bash
mkdir -p public
echo "This is a public practice file." > public/notice.txt
```

Start the server on the loopback interface only:

``` bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Keep this terminal open. Your local target is:

``` text
http://127.0.0.1:8000
```

In a second terminal, confirm that it works:

``` bash
curl -i http://127.0.0.1:8000
```

You should see an HTTP response and the page HTML.

> Python's simple HTTP server is for this learning exercise, not
> production hosting.

## 3. Install the tools on Kali Linux

Update Kali's package information:

``` bash
sudo apt update
```

Install commonly packaged tools:

``` bash
sudo apt install -y nmap whatweb nikto gobuster dirb wafw00f sslscan
```

Check installed versions:

``` bash
nmap --version
whatweb --version
nikto -Version
gobuster version
dirb
wafw00f --version
sslscan --version
```

Some tools may have different version flags. If a command is
unavailable, check the package name and official documentation for your
Kali release.

### Install FFUF

On Kali, try:

``` bash
sudo apt install -y ffuf
```

Check:

``` bash
ffuf -V
```

### Install testssl.sh

If the Kali package is available:

``` bash
sudo apt install -y testssl.sh
```

Check:

``` bash
testssl.sh --version
```

### Install Nuclei

On Kali, first check package availability:

``` bash
apt-cache policy nuclei
```

If available:

``` bash
sudo apt install -y nuclei
```

Check:

``` bash
nuclei -version
```

Use the official project installation instructions if your distribution
does not provide the package.

## 4. Understand a URL and web target

Example URL:

``` text
https://example.test:8443/blog/page
```

  Part             Meaning
  ---------------- ----------
  `https`          Protocol
  `example.test`   Hostname
  `8443`           Port
  `/blog/page`     Path

For your local exercise:

-   Host: `127.0.0.1`
-   Port: `8000`
-   Protocol: HTTP
-   URL: `http://127.0.0.1:8000`

The following commands use only this local target unless an example is
clearly marked otherwise.

## 5. Nmap

**Nmap** is a network discovery and port-scanning tool. It helps
identify which services are listening on a system.

### Check the local web port

``` bash
nmap -p 8000 127.0.0.1
```

Meaning:

-   `nmap`: start Nmap.
-   `-p 8000`: scan only port 8000.
-   `127.0.0.1`: scan your own computer.

### Check a small set of common ports on your own machine

``` bash
nmap -p 22,80,443,8000 127.0.0.1
```

### Detect service information on the local practice port

``` bash
nmap -sV -p 8000 127.0.0.1
```

`-sV` asks Nmap to identify service/version information. Results are
estimates, not guaranteed facts.

### Save output to a text file

``` bash
nmap -sV -p 8000 -oN nmap-local.txt 127.0.0.1
```

Useful options:

  Option           Meaning
  ---------------- -------------------------------------------
  `-p`             Select ports
  `-sV`            Probe services for version information
  `-oN file.txt`   Save normal output to a file
  `-oX file.xml`   Save XML output
  `-Pn`            Skip host discovery; treat host as online

Start with a small, known scope. Do not use broad scans against networks
you do not own.

## 6. WhatWeb

**WhatWeb** identifies technologies used by websites, such as web
servers, content management systems, and frameworks.

### Scan your local practice website

``` bash
whatweb http://127.0.0.1:8000
```

### Increase detail slightly

``` bash
whatweb -v http://127.0.0.1:8000
```

### Save output

``` bash
whatweb --log-verbose=whatweb-local.txt http://127.0.0.1:8000
```

WhatWeb may not identify much on a simple Python server. That is normal.
A scanner can only identify technologies that are visible from its
observations.

## 7. Nikto

**Nikto** checks web servers for various known issues, potentially risky
files, and configuration problems.

### Scan your local practice server

``` bash
nikto -host http://127.0.0.1:8000
```

### Save a report

``` bash
nikto -host http://127.0.0.1:8000 -output nikto-local.txt
```

### Understand the result

Nikto may report missing security headers or paths that deserve
investigation. A missing header is not automatically an exploitable
vulnerability. Verify whether the finding matters for the application.

Nikto can send many requests. Keep it on your local lab or an explicitly
approved target, and respect test limits.

## 8. Gobuster

**Gobuster** is a command-line tool that can discover directories and
files using a wordlist.

### What is a wordlist?

A wordlist is a text file containing possible names to test. For
example:

``` text
admin
images
uploads
public
backup
```

Kali often includes wordlists under:

``` text
/usr/share/wordlists/
```

Check available lists:

``` bash
ls /usr/share/wordlists/
```

The `dirb` package commonly includes a list under:

``` text
/usr/share/dirb/wordlists/common.txt
```

### Find paths on your local practice server

``` bash
gobuster dir \
  -u http://127.0.0.1:8000 \
  -w /usr/share/dirb/wordlists/common.txt \
  -t 5
```

Explanation:

  Option   Meaning
  -------- -----------------------------------
  `dir`    Use directory/file discovery mode
  `-u`     Target URL
  `-w`     Wordlist file
  `-t 5`   Use five concurrent threads

### Include common file extensions

``` bash
gobuster dir \
  -u http://127.0.0.1:8000 \
  -w /usr/share/dirb/wordlists/common.txt \
  -x txt,html \
  -t 5
```

`-x` adds file extensions to the names being checked.

### Save results

``` bash
gobuster dir \
  -u http://127.0.0.1:8000 \
  -w /usr/share/dirb/wordlists/common.txt \
  -t 5 \
  -o gobuster-local.txt
```

Gobuster versions differ. If an option is not recognized, read:

``` bash
gobuster dir --help
```

A path being discovered does not mean it is vulnerable. Check whether it
is intended to be public and whether access controls are correctly
enforced.

## 9. Dirb

**Dirb** is another content-discovery tool. It sends requests for
candidate paths from a wordlist.

### Scan your local practice website

``` bash
dirb http://127.0.0.1:8000 /usr/share/dirb/wordlists/common.txt
```

### Save the output

``` bash
dirb http://127.0.0.1:8000 /usr/share/dirb/wordlists/common.txt -o dirb-local.txt
```

### Gobuster versus Dirb

  -----------------------------------------------------------------------
  Gobuster                            Dirb
  ----------------------------------- -----------------------------------
  Supports multiple modes             Focuses on web content discovery

  Offers configurable concurrency     Has its own request behavior

  Uses `-u` and `-w` in directory     Commonly accepts URL and wordlist
  mode                                as positional arguments

  Good for configurable workflows     Useful for straightforward wordlist
                                      discovery
  -----------------------------------------------------------------------

Do not run both at high speed against the same production system. Choose
one tool and follow the approved request limits.

## 10. FFUF

**FFUF** means Fuzz Faster U Fool. It is a fast web fuzzer that can test
input positions using a wordlist.

For beginners, use it for simple path discovery on the local practice
site.

### Discover paths

``` bash
ffuf \
  -u http://127.0.0.1:8000/FUZZ \
  -w /usr/share/dirb/wordlists/common.txt \
  -rate 5
```

`FUZZ` is the position that FFUF replaces with each word from the
wordlist.

### Save results as JSON

``` bash
ffuf \
  -u http://127.0.0.1:8000/FUZZ \
  -w /usr/share/dirb/wordlists/common.txt \
  -rate 5 \
  -of json \
  -o ffuf-local.json
```

### Useful options

  Option       Meaning
  ------------ ----------------------------------------
  `-u`         Target URL containing the fuzz keyword
  `-w`         Wordlist
  `-rate`      Maximum requests per second
  `-of json`   JSON output format
  `-o`         Output file

Read all available options:

``` bash
ffuf -h
```

FFUF also supports other fuzzing positions, such as headers and request
data. Learn those only in a controlled lab with test accounts and
harmless inputs. Do not use it to guess passwords or access other users'
accounts.

## 11. Wafw00f

A **Web Application Firewall (WAF)** is a security control that filters
or monitors web requests.

**Wafw00f** attempts to identify whether a website uses a known WAF.

### Check your local server

``` bash
wafw00f http://127.0.0.1:8000
```

Your simple Python server probably has no WAF. That is expected.

### Save results

``` bash
wafw00f -o waf-local.txt http://127.0.0.1:8000
```

A WAF identification result is not a full security assessment. A WAF
also does not replace secure application code, authentication,
authorization, and patching.

## 12. SSLScan and testssl.sh

HTTPS uses TLS to protect data in transit. These tools help review TLS
configuration.

The local Python server in this guide uses HTTP, not HTTPS. Therefore,
the TLS commands below are examples for a separate HTTPS service that
you own or are explicitly authorized to test.

### SSLScan

``` bash
sslscan --no-colour your-authorized-host.example:443
```

Replace the example hostname only with a hostname in your approved
scope.

### testssl.sh

``` bash
testssl.sh --fast your-authorized-host.example:443
```

### What to look for

-   Old or weak TLS protocol versions.
-   Weak cipher suites.
-   Certificate expiration.
-   Certificate hostname mismatch.
-   Incomplete certificate chains.
-   Incorrect TLS configuration.

Do not assume a single warning proves that a site is exploitable. Review
the certificate, server configuration, client requirements, and current
security guidance.

## 13. Nuclei

**Nuclei** is a template-based scanner. Templates describe checks for
known technologies, exposures, misconfigurations, and vulnerabilities.

Because templates can make many requests and some checks may be
intrusive, use only a lab or an approved scope.

### Check the version

``` bash
nuclei -version
```

### Update templates

``` bash
nuclei -update-templates
```

### Run a low-impact template scan against your local lab

``` bash
nuclei -u http://127.0.0.1:8000 -t http/technologies/ -rl 2
```

This command uses the HTTP technology templates directory if it exists
in your installed template collection. Template directory names can
change; inspect your installed templates and the official documentation.

### Save output

``` bash
nuclei \
  -u http://127.0.0.1:8000 \
  -t http/technologies/ \
  -rl 2 \
  -o nuclei-local.txt
```

### Important Nuclei safety notes

-   Read the template before running it.
-   Start with informational or low-impact checks.
-   Use conservative rate limits.
-   Avoid intrusive, fuzzing, DoS, and destructive templates.
-   Do not run broad template collections against systems outside your
    written scope.
-   Verify each result manually.

## 14. A safe scanning workflow

For an authorized web application assessment, organize your work in
stages.

### Stage 1: Confirm scope

Record:

-   Approved hostname or IP.
-   Approved ports.
-   Allowed testing methods.
-   Request-rate limits.
-   Testing dates and times.
-   Emergency contact and stop conditions.

### Stage 2: Check connectivity and ports

For a local lab:

``` bash
nmap -p 8000 127.0.0.1
```

### Stage 3: Identify web technologies

``` bash
whatweb http://127.0.0.1:8000
```

### Stage 4: Review common server issues

``` bash
nikto -host http://127.0.0.1:8000
```

### Stage 5: Discover public paths

Choose one tool:

``` bash
gobuster dir -u http://127.0.0.1:8000 -w /usr/share/dirb/wordlists/common.txt -t 5
```

Or:

``` bash
dirb http://127.0.0.1:8000 /usr/share/dirb/wordlists/common.txt
```

Or:

``` bash
ffuf -u http://127.0.0.1:8000/FUZZ -w /usr/share/dirb/wordlists/common.txt -rate 5
```

### Stage 6: Review HTTPS

Only if your authorized target has HTTPS:

``` bash
sslscan your-authorized-host.example:443
```

### Stage 7: Review findings

Confirm whether a result is real, determine its impact, and recommend a
fix. Do not attempt exploitation merely because a scanner reports a
possible issue.

## 15. How to understand results

### HTTP status codes

  Code            Meaning
  --------------- ------------------------------------
  `200 OK`        Request succeeded.
  `301` / `302`   Redirect.
  `400`           Bad request.
  `401`           Authentication required or failed.
  `403`           Server refused access.
  `404`           Resource not found.
  `429`           Too many requests.
  `500`           Internal server error.

A `403` response does not mean the scanner has bypassed access control.
A `404` does not prove a path is absent in every application
configuration.

### Common scanner result terms

-   **Open port:** A service accepted a connection on that port.
-   **Fingerprint:** An estimate of a technology or service based on
    observable behavior.
-   **Potential finding:** A result that needs manual confirmation.
-   **False positive:** A reported issue that is not actually present.
-   **False negative:** A real issue that the scanner did not detect.
-   **Informational:** A detail that may help understand the target but
    is not necessarily a vulnerability.

### Make a simple report

  Field            Example
  ---------------- --------------------------------------------------
  Finding          Missing security header
  Asset            Local practice server
  Evidence         Response did not include the selected header
  Risk             Depends on application behavior and threat model
  Recommendation   Configure appropriate security headers
  Status           Needs review / confirmed / fixed

Do not put passwords, private keys, session cookies, personal data, or
other secrets into a public GitHub report.

## 16. Common problems

### `command not found`

Check installation:

``` bash
which gobuster
which dirb
which ffuf
```

Install the missing Kali package if available:

``` bash
sudo apt update
sudo apt install -y gobuster dirb ffuf
```

### Wordlist does not exist

Check the path:

``` bash
ls -l /usr/share/dirb/wordlists/common.txt
```

If missing, reinstall the package:

``` bash
sudo apt install --reinstall dirb
```

### Connection refused

Check that your Python server is running:

``` bash
curl -i http://127.0.0.1:8000
```

Confirm the port and address:

``` bash
ss -ltn
```

### Too many requests or HTTP 429

Stop the scan and review the authorized rate limit. Reduce concurrency
or request rate. Do not attempt to bypass a site's rate limiting.

### Gobuster reports many results

Some applications return the same status or response size for
nonexistent paths. This is called a wildcard response. Read Gobuster's
help and documentation to understand how to identify and filter these
responses. Do not assume every reported path exists.

## 17. Practice exercises

Complete these exercises against the local Python server from Section 2.

### Beginner

-   [ ] Run `curl -i http://127.0.0.1:8000`.
-   [ ] Scan port 8000 with Nmap.
-   [ ] Identify the local server with WhatWeb.
-   [ ] Run Nikto and save the report.
-   [ ] Explain the difference between a port and a URL path.

### Directory discovery

-   [ ] Run Gobuster using the supplied common wordlist.
-   [ ] Run Dirb against the same local server.
-   [ ] Run FFUF with `FUZZ` in the URL.
-   [ ] Find the practice `public/` directory.
-   [ ] Explain why a discovered path is not automatically a
    vulnerability.

### Intermediate

-   [ ] Read the official documentation for each tool.
-   [ ] Set up an isolated virtual lab.
-   [ ] Review TLS on an HTTPS service you own.
-   [ ] Run only low-impact Nuclei templates against your lab.
-   [ ] Write a short report with evidence, impact, and remediation.

## 18. Quick command reference

All commands in this table use the local practice server unless
otherwise stated.

  --------------------------------------------------------------------------------------------------------------------------
  Tool                                Command
  ----------------------------------- --------------------------------------------------------------------------------------
  Curl                                `curl -i http://127.0.0.1:8000`

  Nmap                                `nmap -sV -p 8000 127.0.0.1`

  WhatWeb                             `whatweb http://127.0.0.1:8000`

  Nikto                               `nikto -host http://127.0.0.1:8000`

  Gobuster                            `gobuster dir -u http://127.0.0.1:8000 -w /usr/share/dirb/wordlists/common.txt -t 5`

  Dirb                                `dirb http://127.0.0.1:8000 /usr/share/dirb/wordlists/common.txt`

  FFUF                                `ffuf -u http://127.0.0.1:8000/FUZZ -w /usr/share/dirb/wordlists/common.txt -rate 5`

  Wafw00f                             `wafw00f http://127.0.0.1:8000`

  Nuclei                              `nuclei -u http://127.0.0.1:8000 -t http/technologies/ -rl 2`
  --------------------------------------------------------------------------------------------------------------------------

TLS tools require an authorized HTTPS target; they are not applicable to
the HTTP-only local server in this exercise.

## 19. Official resources

-   [Nmap Reference Guide](https://nmap.org/book/man.html)
-   [WhatWeb GitHub](https://github.com/urbanadventurer/WhatWeb)
-   [Nikto GitHub](https://github.com/sullo/nikto)
-   [Gobuster GitHub](https://github.com/OJ/gobuster)
-   [Dirb project](https://dirb.sourceforge.net/)
-   [FFUF GitHub](https://github.com/ffuf/ffuf)
-   [Wafw00f GitHub](https://github.com/EnableSecurity/wafw00f)
-   [SSLScan GitHub](https://github.com/rbsec/sslscan)
-   [testssl.sh GitHub](https://github.com/testssl/testssl.sh)
-   [Nuclei GitHub](https://github.com/projectdiscovery/nuclei)
-   [OWASP Web Security Testing
    Guide](https://owasp.org/www-project-web-security-testing-guide/)
-   [OWASP Top 10](https://owasp.org/www-project-top-ten/)

------------------------------------------------------------------------

## Final advice

Learn what each command does instead of running every scanner blindly.
Start with your local lab, keep the request rate low, read the output,
verify findings, and document fixes.

**Only test systems you own or are explicitly authorized to assess.**
