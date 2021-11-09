```bash

awk '{ printf "echo \"%s\" >> /tmp/182079_ifn.o\n", $0 }'  /var/www/html/pee.sh > echo_chamber
```
