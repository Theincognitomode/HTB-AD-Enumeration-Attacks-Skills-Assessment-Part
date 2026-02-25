Q1. Submit the contents of the flag.txt file on the administrator Desktop of the web server 

Hint for us:
Before switching projects, our teammate left a password-protected web shell (with the credentials: admin:My_W3bsH3ll_P@ssw0rd!) in place for us to start from in the /uploads directory. As part of this assessment, our client, Inlanefreight, has authorized us to see how far we can take our foothold and is interested to see what types of high-risk issues exist within the AD environment. 

Lets start the machine and visit the `uploads` folder...

http://ip/uploads, there we can find the "antax.aspx" -> login with `admin:My_W3bsH3ll_P@ssw0rd!` it gives us "A Webshell which utilizes PowerShell", now we can directl
check for the file or what we can do is we can get a stable revshell which can help us futher to enumerate better:

*Lets Get a Stable Rev shell* -> 
