## Foreword:
Reconnaissance being the first step of every web application pentest, it's a repetitive process that's to be done in a systematic way. Thus, it's most beneficial to try automating it as much as possible. So, it turns out that in just about 70 lines of code, we might simply achieve that very objective!

## What is this all about?
Given a domain name, the code below automates the reconnaissance process of web hacking. It simply collects various information about the target domain name. That includes (but not limited to): 
* Subdomains
* Open ports
* SSL ciphers
* HTTP banner
* Directories
* SPF records
* WHOIS records
* WAFs used (if any)
* Subnet active hosts
* Unprotected config files
* Frameworks used (if any)
* Known vulnerabilities (e.g. [Shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug)), [Heartbleed](https://en.wikipedia.org/wiki/Heartbleed))

## Dependencies:
###### All dependencies come preinstalled on Kali Linux 1.0 and later versions.
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
Automates the reconnaissance process of web hacking.
"""
import sys
import socket
import subprocess
from time import sleep

def main():
    """ Executes main code """
    domain = sys.argv[1]
    try:
        ip_address = socket.gethostbyname(domain)
    except socket.gaierror:
        print 'Error: Domain name cannot be resolved!'
        raise
    procs = []
    whois_cmd = ['whois', domain]
    dig_cmd = ['dig', '-t', 'txt', '+short', domain]
    wpscan_cmd = ['wpscan', '--force', '--update', '--url', domain]
    nmap_hosts_cmd = ['nmap', '-sn', ip_address + '/24']
    nmap_script_names = ('dns-brute, hostmap-ip2hosts, banner,'
                         'http-robots.txt, http-crossdomainxml, http-enum,'
                         'http-config-backup, http-devframework, http-methods,'
                         'http-waf-fingerprint, http-sitemap-generator,'
                         'http-xssed, http-shellshock, ftp-anon, ssl-cert,'
                         'ssl-poodle, ssl-heartbleed, ssl-enum-ciphers')
    nmap_full_cmd = ['nmap', '-sV', '-sS', '-A', '-Pn', '--script',
                     nmap_script_names, domain]
    cmds = {'TXT Records': dig_cmd, 'WHOIS Info': whois_cmd,
            'Nmap Results': nmap_full_cmd, 'Active Hosts': nmap_hosts_cmd,
            'WPScan': wpscan_cmd}

    def handle_proc(proc):
        """ handles subprocesses outputs """
        separator = '=================='
        output = ''.join(proc.stdout.readlines())
        print proc.title
        print separator
        print output.strip()
        print separator + '\n'
        procs.remove(proc)

    for title, cmd in cmds.items():
        try:
            proc = subprocess.Popen(cmd, stdout=subprocess.PIPE)
            proc.title = title
            procs.append(proc)
        except OSError:
            print '%s >> Dependency error occurred!\n' % title

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
    print "This is gonna take quite a while; you better go make some coffee!\n"
    main()

```

## Screenshot(s):
[![terminal.jpg](https://s3.postimg.org/fg0j4bi8j/terminal.jpg)](https://s3.postimg.org/fg0j4bi8j/terminal.jpg)
[![AutoRecon.png](https://s14.postimg.org/nq01wywdt/Auto_Recon.png)](https://postimg.org/image/ei7tg9pbh/)
## Bottom Line:
If you find the stuff in this repo very helpful, you may (graciously) consider passing some bitcoin goodies to this address `14FZTgfX4QmDGUoXYpSUdKFJEjbvQnArAu`.