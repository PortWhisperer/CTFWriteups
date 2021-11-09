# Metasploit Scripts
Scripts that support Metasploit operations 

[Echo Chamber](echo_chamber.md)- msf `session -s ` script runner doesn't support bash scripts natively. Use Echo chamber to force script handler to pipe commands into file on remote host, then allow remote hosts shell environment to execute the output.