# Scope
172.16.1.0/24 - 172.16.17.0/24

# Obfuscation
said that obfuscation may not be required for the exam, you can either:
- look for exception paths for Defender
- obfuscate the tools yourself
all obfuscated tools are give in the labs



# BloodHound
provided VM does not have enough resources to run BloodHound, so they have a shared instance with read-only access
- https://crtpbloodhound-altsecdashboard.msappproxy.net/
	- credentials can be found in lab portal
- see video 08 for more details


# Miscellaneous
ALWAYS target active users
- when looking as user properties, check logon count -- if near or at zero, could be a honeypot