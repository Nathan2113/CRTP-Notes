# Tools

PowerShell
`Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll`
`Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1`

[BloodHound](https://github.com/BloodHoundAD/BloodHound) (C# and PowerShell Collectors)

[PowerView](https://github.com/ZeroDayLab/PowerSploit/blob/master/Recon/PowerView.ps1) (PowerShell)

[SharpView (C#)](https://github.com/tevora-threat/SharpView/) - Doesn't Support filtering using Pipeline


# PowerView and PowerShell
- PowerView is first command, PowerShell is second

Get Domain Information
`Get-Domain`
`Get-ADDomain`

Get Domain SID
`GetDomainSID`
`(Get-ADDomain).DomainSID`

Get object of another domain
`Get-Domain -Domain <DOMAIN>`
`Get-ADDomain -Identity <DOMAIN>`

Get Domain policy (Kerberos, password, etc)
`Get-DomainPolicyData`
`(Get-DomainPolicyData).systemaccess`

Get Domain policy for another Domain
`(Get-DomainPolicyData -domain <DOMAIN>).systemaccess`

Get Domain controllers for current Domain
`Get-DomainController`
`Get-ADDomainController`

Get Domain controllers for another Domain
`Get-DomainController -Domain <DOMAIN>`
`Get-ADDomainController -DomainName <DOMAIN> -Discover`

Get a list of users in the current Domain
`Get-DomainUser`
`Get-DomainUser -Identity <username>`
`Get-ADUser -Filter * -Properties *`
`Get-ADUser -Identity <username> -Properties *`

Get list of all properties for users in the current domain
`Get-DomainUser -Identity <username> -Properties *`
`Get-DomainUser -Properties samaccountname,logonCount`
`Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -Memberype *Property | select Name`
`Get-ADUser -Filter * -Properties * | select name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}`

Search for a particular string in a user's attributes
`Get-DomainUser -LDAPFilter "Description=*built*" | Select name, Description`
`Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Description`

Get a list of computers in the current domain
`Get-DomainComputer | select dnshostname,logonCount`
`Get-DomainComputer -OperatingSystem "*Server 2022*"`
`Get-DomainComputer -Ping`
`Get-ADComputer -Filter * | select dnshostname,logonCount`
`Get-ADComputer -Filter * -Properties *`
`Get-ADComputer -Filter 'OperatingSystem -like "*Server 2022*"' -Properties OperatingSystem | select Name,OperatingSystem`
`Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerNmae $_.DNSHostName}`

Get all the groups in the current domain
`Get-DomainGroup | select Name`
`Get-DomainGroup -Domain <targetdomain>`
`Get-ADGroup -Filter * | select Name`
`Get-ADGroup -Filter * -Properties *`

Get all groups containing the word "admin" in group name
`Get-DomainGroup *admin*`
`Get-ADGroup -Filter 'Name -like "*admin*"' | select Name`

Get all members of the domain admins group
`Get-DomainGroupMember -Identity "Domain Admins" -Recurse`
`Get-ADGroupMember -Identity"Domain Admins" -Recursive`

Get the group membership for a user
`Get-DomainGroup -UserName <username>`
`Get-ADPrincipalGroupMembership -Identity <username>`

List all local groups on a machine (needs administrator privs on non-dc machines)
`Get-NetLocalGroup -ComputerName <NetBIOS>`

Get members of local group "Administrators"a machine
`Get-NetLocalGroupMember -ComputerName <NetBIOS> -GroupName Administrators`

Get actively logged on users (needs admin)
`Get-NetLoggedon -ComputerName <computer>`

Get locally logged users on a computer (needs remote registry on target - default on server)
`Get-LoggedonLocal -ComputerName <computer>`

Get the last logged on user on a computer (needs admin and remote registry)
`Get-LastLoggedon -ComputerName <computer>`

Find shares on hosts in domain
`Invoke-ShareFinder -Verbose`


Find sensitive files on computers in domain
`Invoke-FileFinder -Verbose`

Get all fileservers of the domain
`Get-NetFileServer`

For enumerating shares, can also use [PowerHuntShares](https://github.com/NetSPI/PowerHuntShares) 
`Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools -HostList C:\AD\Tools\servers.txt`
- servers.txt does not include the domain controller for better OPSEC
- includes ShareGraph that can be used to explore share relationships