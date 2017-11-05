## Foreword:
Reconnaissance being the first step of every web application pentest, it's a repetitive process that's to be done in a systematic way. Thus, it's most beneficial to try automating it as much as possible. So, it turns out that in just about 70 lines of code, we might simply achieve that very objective!

## What is this all about?
Given a domain name, the code below automates the reconnaissance process of web hacking. It simply collects various information about the target domain name. That includes (but not limited to): 
* Subdomains
* Open ports
* Directories
* SSL ciphers
* SPF records
* WHOIS records
* Services' banners
* WAFs used (if any)
* Subnet active hosts
* Unprotected config files
* Frameworks used (if any)
* Known vulnerabilities (e.g., [Shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug)), [Heartbleed](https://en.wikipedia.org/wiki/Heartbleed), el al.)

## Dependencies:
###### All dependencies come preinstalled on Kali Linux 1.x and later versions.
* [DIG](https://en.wikipedia.org/wiki/Dig_(command))
* [Nmap](https://nmap.org)
* [WHOIS](https://en.wikipedia.org/wiki/WHOIS#Software)
* [WPScan](https://wpscan.org)

## How to use:
Open your terminal (preferably from Kali Linux) and execute the script below using this command:
```shell
sudo python AutoRecon.py example.com
```
###### P.S. Root privileges are required. Additionally, if you'd like to save the output into a file, you may use `sudo python AutoRecon.py example.com >> output.txt`.

## Code:
```python
#!/usr/bin/env python2
# -*- coding: utf-8 -*-
"""
Automate the reconnaissance process of web hacking.
"""
import sys
import socket
import subprocess
from time import sleep

def main():
    """Execute main code."""
    try:
        domain = sys.argv[1]
        ip_address = socket.gethostbyname(domain)
    except IndexError:
        print 'Error: Domain name not specified.'
        sys.exit(1)
    except socket.gaierror:
        print 'Error: Domain name cannot be resolved.'
        raise
    procs = []
    whois_cmd = ['whois', domain]
    dig_cmd = ['dig', '-t', 'txt', '+short', domain]
    wpscan_cmd = ['wpscan', '--force', '--update', '--url', domain]
    nmap_hosts_cmd = ['nmap', '-sn', ip_address + '/24']
    nmap_script_names = ('*-vuln*, banner, default, dns-brute,'
                         'dns-zone-transfer, ftp-*, hostmap-ip2hosts, http-config-backup,'
                         'http-cross*, http-devframework, http-enum, http-headers,'
                         'http-shellshock, http-sitemap-generator, http-waf-fingerprint,'
                         'http-xssed, smtp-*, ssl-*, version')
    nmap_full_cmd = ['nmap', '-sV', '-sS', '-A', '-Pn', '--script',
                     nmap_script_names, domain]
    cmds = {'TXT Records': dig_cmd, 'WHOIS Info': whois_cmd,
            'Active Hosts': nmap_hosts_cmd, 'Nmap Results': nmap_full_cmd,
            'WPScan': wpscan_cmd}

    def handle_proc(proc):
        """Handle subprocesses outputs."""
        separator = '=================='
        output = ''.join(proc.stdout.readlines())
        print proc.title
        print separator
        print output.strip()
        print separator + '\n'
        procs.remove(proc)

    for title, cmd in cmds.items():
        try:
            proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
            proc.title = title
            procs.append(proc)
        except OSError:
            print '%s >> Dependency error occurred.\n' % title

    while True:
        for proc in procs:
            retcode = proc.poll()
            if retcode is not None:
                handle_proc(proc)
            else:
                continue
        if len(procs) == 0:
            break
        else:
            sleep(1)

if __name__ == '__main__':
    print 'This is gonna take quite a while; you better go make some coffee!\n'
    main()

```

## Screenshot(s):
[![terminal.png](https://s27.postimg.org/jd3xoyl6r/terminal.png)](https://postimg.org/image/8qa4jjd1b/)
[![terminal2.png](https://s28.postimg.org/5jakqmwal/terminal2.png)](https://postimg.org/image/upbixgxkp/)
## Footnote:
If you find the stuff in this repo very helpful, you may (graciously) consider passing some bitcoin goodies to this address `14FZTgfX4QmDGUoXYpSUdKFJEjbvQnArAu`.