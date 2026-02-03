Find all machines on the current domain where the current user has local admin access
`Find-LocalAdminAcceess -Verbose`

- queries the DC of the current or provided domain for a list of computers (`Get-NetComputer`) and then uses `Invoke-CheckLocalAdminAccess` on each machine
- can be done with remote administration tools like WMI and PowerShell remoting
	- useful when RPC and SMB are blocked
- see `Find-WMILocalAdminAccess.ps1` and `Find-PSRemotingLocalAdminAccess.ps1`
- not silent (leaves 4624 - Logon, 4634 - Logoff, and 4672 - Admin Access)


Find computers where a domain admin (or specific user) has sessions
` Find-DomainUserLocation -Verbose`
`Find-DomainUserLocation -UserGroupIdentity "RDPUsers"`

- queries the DC using `Get-DomainGroupMember`, gets a list of computers (`Get-DomainComputer`), and list sessions and logged on users (`Get-NetSession`/`Get-NetLoggedon`) from each machine
- for servers 2019 onwards, local administrator is required to list sessions
- also noisy (if you forget to remove DC from list of computers it will create an alert)
	- PROVIDE A LIST OF MACHINES AND EXCLUDE DC


List sessions on remote machines using [Invoke-SessionHunter](https://github.com/Leo4j/Invoke-SessionHunter)
`Invoke-SessionHunter -FailSafe`
`Invoke-SessionHunter -NoPortScan -Targets C:\path\to\targets.txt`
- opsec friendly, avoid connecting to all target machines by specifying targets

- doesn't need admin access on remote machines
- uses remote registry and queries HKEY_USERS hive
