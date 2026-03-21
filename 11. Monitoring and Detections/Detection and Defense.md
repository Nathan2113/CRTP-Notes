# Common Defense Recommendations
- protect and limit domain admins
	- don't allow DA's to log into anything besides DCs
	- never run a service with DA
	- set "Account is sensitive and can't be delegated"
- make use of Protected Users Group
	- better protection against credential theft
		- does not cache credentials in insecure ways
			- no CredSSP or WDigest
			- NTLM hash not cached
			- Kerberos does not use DES or RC4
- Privileged Administrative Workstations (PAWs)
	- hardened workstation for performing sensitive tasks like administration of Domain Controllers
	- can provide protection from phishing attacks, OS vulns, credential replay attacks
- LAPS
	- centralized storage of passwords in AD with periodic randomizing
	- storage in clear text, transmission is encrypted
	- with careful enumeration, it is possible to retrieve the users that can access the clear text passwords

