## Foreword:
Reconnaissance being the first step of every web application pentest, it's a repetitive process that's to be done in a systematic way. Thus, it's most beneficial to try automating it as much as possible. So, it turns out that in just about 70 lines of code, we might simply achieve that very objective!

## What is this all about?
Given a domain name, the code below automates the reconnaissance process of web hacking. It simply collects various information about the target domain name. That includes (but not limited to): 
* Subdomains
* Open ports
* Directories
* SPF records
* WHOIS records
* WAFs used (if any)
* Subnet active hosts
* Unprotected config files

## Dependencies:
###### All dependencies come preinstalled on Kali Linux 1.0 and later versions.
* [Nmap](https://nmap.org)
* [DIG](https://en.wikipedia.org/wiki/Dig_(command))
* [WHOIS](https://en.wikipedia.org/wiki/WHOIS#Software)

## How to use:
Open your terminal (preferably from Kali Linux) and execute the script below using this command:
```shell
sudo python AutoRecon.py example.com
```
###### P.S. Root privileges are required.

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
    wpscan_cmd = ['wpscan', '--url', domain]
    dig_cmd = ['dig', '-t', 'txt', '+short', domain]
    nmap_hosts_cmd = ['nmap', '-sn', ip_address + '/24']
    nmap_enum_cmd = ['nmap', '-Pn', '-sn', '--script=http-enum', domain]
    nmap_dnsbrute_cmd = ['nmap', '-Pn', '-sn', '--script=dns-brute', domain]
    nmap_conf_cmd = ['nmap', '-Pn', '-sn',
                     '--script=http-config-backup', domain]
    nmap_ports_cmd = ['nmap', '-sS', '-A', '-Pn', '-p-',
                      '--script=http-title', domain]
    nmap_waf_cmd = ['nmap', '-Pn', '-sn', '-sV',
                    '--script=http-waf-fingerprint', domain]
    cmds = {'Subdomains': nmap_dnsbrute_cmd, 'WHOIS Info': whois_cmd,
            'Open Ports': nmap_ports_cmd, 'Active Hosts': nmap_hosts_cmd,
            'WPScan': wpscan_cmd, 'Directories': nmap_enum_cmd,
            'WAFs': nmap_waf_cmd, 'Config Backups': nmap_conf_cmd,
            'TXT records': dig_cmd}

    for title, cmd in cmds.items():
        try:
            proc = subprocess.Popen(cmd, stdout=subprocess.PIPE)
            proc.title = title
            procs.append(proc)
        except OSError:
            print '%s >> Dependency error occurred!\n' % title

    def handle_proc(proc):
        """ handles active subprocesses """
        separator = '=================='
        output = ''.join(proc.stdout.readlines())
        print proc.title
        print separator
        print output.strip()
        print separator + '\n'
        procs.remove(proc)

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

## Bottom Line:
If you find the stuff in this repo very helpful, you may (graciously) consider passing some bitcoin goodies to this address `14FZTgfX4QmDGUoXYpSUdKFJEjbvQnArAu`.