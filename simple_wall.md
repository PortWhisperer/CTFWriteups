[2020-Oct-04 06:02:07 UTC] > used zsteg to decode challenge
[2020-Oct-04 06:12:14 UTC] > found some semi useful comments on the admininstrator/index.php tab. also found a flag in /images/flag.txt.txt. gained appreciation for nmap --script http-\* particularly the comment enum functionality. would like to build out comment function a bit further
[2020-Oct-09 05:44:06 UTC] > stuck trying to upload shell. form accepts images and server is running php. php rev shell not working. modifying extension to php allows upload, but server complains of errors in the png file. listerner doesnt catch reverse shell.
[2020-Oct-09 05:45:58 UTC] > had success with the zsteg tool after trying close to 10 tools that couldn't identify the stego in hidden.png. embarrassingly actually contacted author of the box, who also suggested zsteg. has been added to arsenal
[2020-Oct-09 06:01:52 UTC] > binary observed in www-data directory contained the strings Digite a senha: MrR0b0t121 n Senha errada n Senha Correta n Password@1238
[2020-Oct-09 06:02:16 UTC] > home directory also contained a dot file indicating the user has sudo rights
