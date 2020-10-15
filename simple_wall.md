Without further ado.

Kicked off the nmap scan

```
# Nmap 7.80 scan initiated Sun Oct  4 02:37:02 2020 as: nmap -p- -sCV -O -A -oA /home/kali/simple_wall/enum/nmap/tcpscan.2020-10-04.02:37:02 172.16.2.42
Nmap scan report for 172.16.2.42
Host is up (0.0015s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Ubuntu 10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 98:b7:f5:6b:0d:58:1d:7b:58:7d:1a:99:fb:b1:8f:04 (RSA)
|   256 66:b4:4b:40:e6:c9:76:93:31:aa:fc:ff:9a:40:a9:f9 (ECDSA)
|_  256 55:c6:b2:01:0f:16:1c:68:96:e2:bb:b1:fe:ff:59:c2 (ED25519)
53/tcp   open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Eskwela Template
5355/tcp open  llmnr?
MAC Address: 08:00:27:ED:95:60 (Oracle VirtualBox virtual NIC)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
```

What these mean  
**22/SSH** is usually not going to net easy wins in CTF's.  
**53/TCP** is also not a usual vector although we have seen a recent CVE for a Windows DNS RCE based on a longtime bug in the OS. Likely a rabbithole to pursue further.  
**80/HTTP**: Web apps are always a rich source of vulns, so this is immediately high priority. We get a version for the apache http daemon which could turn up a vuln as a result of the -sCV tag the (sC triggers some basic NSE scripts)  
**5355/LLMNR** link-local multicast name resolution is a service that supports DNS on local subnets [wikipedia](https://en.wikipedia.org/wiki/Link-Local_Multicast_Name_Resolution). Most likely this is working in conjunction with the DNS service on port 53. It's not immediately clear why the box is running this, and ultimately the exploitation vector had nothing to do with it.

## Attacking TCP/80/HTTP

My typical procedure is to run a directory crawler that supports a proxy such as dirb and then have ZAP or BurpSuite proxy the traffic, causing them to automatically construct a nice site map. In addition, this helps to seed the spidering features on both of those applications:

```
dirb https://172.16.2.42 /usr/share/wordlists/SecLists-master/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" -p http://127.0.0.1:8080

```

\*\*Note\*\* when proxying a scanner like this, make sure you filter 404s out of the site map before starting. If not done, your site map will fill up with 1000s of entries for 404 responses. Burp filters 4xx by default (which could be too much as 403s can be very useful) but ZAP doesn't.

In addition, I picked at random one of the most common/popular user agents. This is to avoid the "script-kiddie" mistake of brute forcing a page and advertising it with a user agent that names your brute forcing tool.

a /administrator page was picked up by the above wordlist. It redirected to an index.php which possessed a login form, which I manually tested for SQL injectability. Injection failed so I attempted to brute force it by testing out some predictable creds and passwords. I also used Cewl to scrape username/password ideas, and made lists with that. All of these failed.

```bash
sqlmap 172.16.2.42/administrator/index.php --forms
```

```bash
cewl 172.16.2.42
```

```bash
for val in (ls /usr/share/wordlists/SecLists-master/Passwords/Common-Credentials/10-million*.txt)
            echo "****************************** $val ******************************"
            hydra -lmrrobot -P $val 172.16.2.42 http-post-form "/administrator/index.php:username=^USER^&pass=^PASS^:F=Failed" -e nsr -t27 -f -I
    end

```

anused zsteg to decode challenge  
[2020-Oct-04 06:12:14 UTC] > found some semi useful comments on the admininstrator/index.php tab. also found a flag in /images/flag.txt.txt. gained appreciation for nmap --script http-\* particularly the comment enum functionality. would like to build out comment function a bit further  
[2020-Oct-09 05:44:06 UTC] > stuck trying to upload shell. form accepts images and server is running php. php rev shell not working. modifying extension to php allows upload, but server complains of errors in the png file. listerner doesnt catch reverse shell.  
[2020-Oct-09 05:45:58 UTC] > had success with the zsteg tool after trying close to 10 tools that couldn't identify the stego in hidden.png. embarrassingly actually contacted author of the box, who also suggested zsteg. has been added to arsenal  
[2020-Oct-09 06:01:52 UTC] > binary observed in www-data directory contained the strings Digite a senha: MrR0b0t121 n Senha errada n Senha Correta n Password@1238  
[2020-Oct-09 06:02:16 UTC] > home directory also contained a dot file indicating the user has sudo rights
