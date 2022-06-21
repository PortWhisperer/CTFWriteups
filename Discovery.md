# Discovery
Discovery related resources by operating system
___
## Linux 
Tactics, Techniques, 
Procedures
Check for recently modified files
- Sort Files Based on Date Accessed
--``` find / -type f -printf "\n%AD %AT %p" | head -n 11```
--sort files modified by specific user in past 90 days
--```find /home -user $USER -mtime -90```
- Check for most recently updated  files, newest first
 --```find /etc -type f -printf '%TY-%Tm-%Td %TT %p\n' | sort -r```
- To search for files in `/target_directory` and all its sub-directories no more than `3` levels deep, that have been modified in the last `2` days:
```find /target_directory -type f -mtime -2 -depth -3```
 modified in the last `7` days, but not in the last `3` days:
 ```$ find /target_directory -type f -mtime -7 ! -mtime -3```
 To search for files in `/target_directory` (and all its sub-directories) that have been modified in the last `60` minutes, and print out their file attributes:
```$ find /target_directory -type f -mmin -60 -exec ls -al {} ;```
or 
```$ find /target_directory -type f -mmin -60 | xargs ls -l```

Tools
___
## Windows
Tactics, Techniques, Procedures
Tools