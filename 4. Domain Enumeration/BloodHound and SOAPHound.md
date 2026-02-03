# BloodHound CE

Supply data to BloodHound
`C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Sharphound\SharpHound.exe -args --collectionmethods All`
- collectionmethods All is going to be a problem and set off alarms

make BloodHound collection stealthy
- use `-ExcludeDCs` flag
- `C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Sharphound\SharpHound.exe -args --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets,CertServices --excludedcs


# SOAPHound

use SOAPHound for more stealth
- talks to ADWS (AD Web Services - Port 9389)
- almost no network-based detection (like MDI)
- retrieves information about all objects (objectGuid=\*) and then processes them, meaning limited LDAP queries

Build a cache that includes basic info about domain objects
`SOAPHound.exe --buildcache -c C:\AD\Tools\cache.txt`

Collect BloodHound compatible data
`SOAPHound.exe -c C:\AD\Tools\cache.txt --bhdump -o C:\AD\Tools\bloodhound-output --nolaps`

