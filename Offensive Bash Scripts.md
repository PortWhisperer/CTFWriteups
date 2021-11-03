# Offensive Bash Scripts

## Reconnaissance
 [[Gobuster Wrapper]] - Automatically selects wordlists and passes them to Gobuster based on desired scan size

## Initial Access
 [[Nmap2searchsploit]] -Sends nmap results to searchsploit and saves output to disk

## Privilege Escalation
[[dpkg2searchsploit]] - (planned) gets all installed packages on linux machine and checks them against searchsploit
[[pslist2searchsploit]] - gets all running proceses on linux machine and checks against searchsploit


## Discovery
[[Linux Discovery Script]] - Downloads several discovery scripts to a linux victim, executes them,  attempts upload back to C2 via curl (SCP support under development), then attempts to destroy output files