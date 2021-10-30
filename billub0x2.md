## Vulnhub Writeup #2: Billu B0x 2
https://www.vulnhub.com/entry/billu-b0x-2,238/

This is my second vulnhub writeup. Things didn't go as well for me with this box - I had some holes in my approach.

We begin with some layer 4 reconnaisance. 


```
nmap -sV -sT -p- [billubox IP] -T5
```

There are two webservers so I used ZAP to enumerate them, first with the spider feature,  then follow that up with a recursive brute force.

The site map for the http server on 80 was rather extensive, as ZAP verbosely includes pages that returned 403's, 302's etc. This muddied the waters substantially. I was able to determine the server was running Drupal, just not what version. There didn't seem to be anything in the source to shed light on that detail, which was a bit frustrating and kept me from enumerating vulnerabilities effectively.

The Tomcat 7 server on 8080 appeared to be a decoy. There was a login form for the management portal, but enumeration didn't turn up much else of use. I did however confirm there were no default creds for Tomcat 7 servers; as a result I set this vector aside: I tend to avoid brute-forcing until I'm certain there are no other vectors into the machine.

I spent some time clicking around the sitemap for both webservers trying to put together some sort of pattern, but I wasn't able to identify anything. I became exasperated and started using Nikto and other scanners to try and find something silly.

Nikto didn't turn anything up, but Whatweb did. This was a pleasant surprise as I'd never used Whatweb before and didn't know what to expect. At any rate, the version of Drupal instance on port 80 was spit out, and checking Searchsploit for Drupal 8 revealed there was a relevant RCE exploit available in Metasploit.

Plugging in the remote host was sufficient to get the module to work, and after running it, we get a low priv shell in the context of www-data.


####privilege escalation

This next part was ratehr exasperating to me, and I ripped some holes in my methodology. I tried to poke around in directories looking for things which were out of place, and I found 2 interesting things. Firstly, having local access to the port 80 webserver allowed me to get credentials for the MySQL backend DB and enumerate this for credentials. I found the SHA512 hash corresponding to the admin user, but wasn't able to crack it even after deploying several million words against it with Hashcat64.

Next I found a hash for user indishell in /etc/passwd. I tried to crack this with several million words from the SecList passwords dump and again failed. At this point I tried to go back through all of the directories and running applications looking for odd things, but turned up nil. 

##### method 1
Finally, I gave up and decided to peek at a walkthrough. It turned out my error was not realizing that /etc/passwd had world writeable permissions. This allows us to set the effective permissions for any user as well as adjust the password of any user. As such, it's possible to generate an /etc/passwd compatible password with the crypt function and overwrite indishell's currently existing hash with it. Next, we simply have to change indishell's UID/GID to 0.

Since I was manually enumerating I missed this obvious vector. If I had used linenum.sh or a similar post exploitation enumeration script, I would have found this with ease.

##### method 2
There's a SUID binary, /opt/s, which runs as root and executes a file, scp, which isn't qualified by a path. Therefore, putting a file named scp into our current directory and making sure that's at the front of our path variable will cause /opt/s to execute our 'scp' file.  The owner of the SUID is root, so this can be leveraged to attain a root shell. 

Lessons learned:
Added Whatweb to the collection of CMS scanning tools I will be using regularly
Use automatic enumeration scripts such as linenum.sh to avoid missing silly things like a world writeable /etc/passwd file. These scripts will also find SUID binaries and format the output in a nice way.

Tags:
#vulnhub #linux