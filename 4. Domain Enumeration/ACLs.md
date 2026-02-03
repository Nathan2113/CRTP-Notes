![[assets/Pasted image 20260111032741.png|Pasted image 20260111032741.png]]
- check marks are covered in the course


Get the ACLs associated with the specified object
`Get-DomainObjectAcl -SamAccountName <user> -ResolveGUIDs`

Get ACLs associated with the specified prefix to be used for search
`Get-DomainObjectAcl -SearchBase "LDAP://CN=Domain Admins,CN=Users,DC=<DOMAIN>,DC=<DOMAIN> -ResolveGUIDs -Verbose`

Enumerate ACLs using ActiveDirectory Module but without resolving GUIDs
`(Get-Acl'AD:\CN=Administrator,CN=Users,DC=<DOMAIN>,DC=<DOMAIN>').Access`

Search for interesting ACEs (PowerView)
`Find-InterestingDomainAcl -ResolveGUIDs`

Get the ACLs associated with the specified path (PowerView)
`Get-PathAcl -Path "\\<computer>\sysvol`
- Example - `Get-PathAcl -Path \\dcorp-dc.dollarcorp.moneycorp.local\sysvol`


