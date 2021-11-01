---
aliases: [Reconnaissance, Recon]
---

# Reconnaissance 

[Bash scripting resources: for-loops](https://linuxhint.com/bash_loop_list_strings/)
### HTTP server dir scanning Script
```bash 
#get vars from user
while getopts 't:s:' OPTION; do
    case "$OPTION" in
        t)
            target="$OPTARG" 
            echo "Targeting $OPTARG"
            ;;
        s)
            scan_size="$OPTARG"
            echo "Scan size selected = $OPTARG"
            ;;
        *)
            echo "Usage: $0 [-target ip] [-size (small|medium|large)] " >&2
            exit 1
        ;;
    esac
done
#select wordlists
declare -a Wordlist_Array=(
        "/usr/share/seclists/Discovery/Web-Content/raft-$scan_size-directories.txt" 
        "/usr/share/seclists/Discovery/Web-Content/raft-$scan_size-files.txt"
        "/usr/share/wordlists/dirb/small.txt")

case $scan_size in
        small)
            Wordlist_Array+=("/usr/share/wordlists/dirb/small.txt")
            ;;
        medium)
            Wordlist_Array+=("/usr/share/wordlists/dirb/common.txt")
            ;;
        large)
            Wordlist_Array+=("/usr/share/wordlists/dirb/big.txt")
            ;;
esac
declare -a Wordlist_Array=("/usr/share/seclists/Discovery/Web-Content/raft-$scan_size-directories.txt" 
"/usr/share/seclists/Discovery/Web-Content/raft-$scan_size-files.txt")

#begin gobuster scan
for i in "${Wordlist_Array[@]}";
do gobuster -w $i dir -u $target -o $target-gobuster_output.txt | egrep "(2|3|4|5)[0-9]{2}"|grep -vi "404"

done
#cat "$target-gobuster_output.txt"
#egrep "(200|4\d{2})"|grep -vi "404" ##//watch for dirs/files with 404 in name
```


Tags: #Reconnaissance #Scripts
