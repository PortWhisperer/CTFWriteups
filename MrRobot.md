

```
wpscan --url http://172.16.2.37 --enumerate vp
```


```
hydra 172.16.2.37 -l Elliot -P sortfs.dic http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&rememberme=forever&redirect_to=http%3A%2F%2F172.16.2.37%2Fwp-admin%2Fupdate-core.php&testcookie=1&wp-submit=Log+In:incorrect" -V | grep -v ATTEMPT
```

https://github.com/wetw0rk/malicious-wordpress-plugin/blob/master/wordpwn.py


upload the zip generated by wordpwn.py to the plugin section of the wordpress control panel

Going to the plugin editor section and selecting the "Got Em" plugin, we're able to see details about the malicious resources we uploaded.
Taking the path specified, and then scanning ZAP's history of HTTP responses from the server for mentions of "plugin" we're able to find the root plugin directory. Combining the root plugin directory with the details of the malicious worw0rk_maybe.php file, we construct a get request to the malicious resource and trigger a reverse shell connection.

Prior to this, a meterpreter listener was setup to listen for a php/meterpreter/reverse_tcp payload on port 4444 using multi/handler module in MSF.
The shell is captured without much fanfare. 

Next steps are to enumerate the machine. I have a set of linux enumeration scripts including linenum.sh, linuxprivchecker.py.

running these, we see there is an nmap binary owned by root with the suid bit set. This allows us to run nmap --interactive, and in the resulting shell, issue commands with the syntax ```![command]```. 

running ```!whoami```  in this context confirms we're root. Running /bin/dash  returns an interactive root shell:


