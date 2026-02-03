Get list of GPOs in the current domain
`Get-DomainGPO`
`Get-DomainGPO -ComputerIdentity <computer>`

Get GPOs which use Restricted Groups or groups.xml for interesting users
`Get-DomainGPOLocalGroup`

cannot check GPOs for another machine remotely

Get users which are in a local group of a machine using a GPO
`Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity <computer>`

Get machines where the given user is a member of a specific group
`Get-DomainGPOUserLocalGroupMapping -Identity <user> -Verbose`

List OUs in a domain
`Get-DomainOU`
`Get-ADOrganizationalUnit -Filter * -Properties *`

Get GPO applied on an OU. Read GPOname from gplink attribute from Get-NetOU
`Get-DomainGPO -Identity "<group_policy>"`
- group policy can be found as the long string between curly braces when looking at OUs
![[assets/Pasted image 20260111040942.png|Pasted image 20260111040942.png]]
![[assets/Pasted image 20260111041013.png|Pasted image 20260111041013.png]]


# Learning Objective

list all OUs
`Get-DomainOU | select -ExpandProperty name`

list all computers in the DevOps OU
`(Get-DomainOU -Identity DevOps).distinguishedname | %(Get-DomainComputer -SearchBase $_} | select name`

list all GPOs
`Get-DomainGPO`

enumerate GPO applied on the DevOps OU
`Get-DomainGPO -Identity "{0BF8D01C-1F62-958C-57140B67D147}"`

enumerate ACLs for the Applocker and DevOps GPOs
![[assets/Pasted image 20260111041614.png|Pasted image 20260111041614.png]]
