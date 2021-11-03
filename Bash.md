## Nmap2searchsploit
Sends nmap results to searchsploit and saves output to disk
```bash
# script expects to cat nmap output from file 
IFS=$'\n'; # instruct bash to use \n as the field separator
victim=172.16.2.44; 
counter=1 
for nmap_outf in $(ls *nmap_output*);
 do cat $nmap_outf | grep -vi "filtered" |grep -E "[[:digit:]]+\/"|cut -b 22-|cut -d "(" -f1 > nmap_applications;
 for app_fullname in $(cat nmap_applications);
  do app_shortname=$(echo $app_fullname|cut -d " " -f1); outfile=${victim}_searchsploit_app_${counter}-${app_shortname}.txt;
 banner=$(printf '#%.0s' $(seq 1 $(tput cols)));
 echo "${banner}Searchsploit results for ${app_fullname}running on $victim\n${banner}\n" > $outfile;
 /bin/bash searchsploit remote $app_shortname>>$outfile; let counter++;done;
done;
rm nmap_applications


```
#searchsploit #automation