---
title: HTB Writeup - Search
date: 2022-02-17T11:26:23.264Z
image: /images/2022-02-10_14-10.png
tags:
  - hackthebox
  - windows
  - search
  - activedirectory
  - kerberoasting
  - sheetProtection
  - bloodhound
  - GMSA
  - ReadGMSAPassword
  - GenericAll
  - crackmapexec
draft: false
---
### Machine Details
#### Name: Search
#### OS: Windows
#### IP address: 10.10.11.129
#### Difficulty: Hard
#### Points: 40

---

### Initial Enumeration
As always, I started with an nmap scan to find out what services are running on the box.
![](https://i.imgur.com/Xc4DH28.png)

From LDAP enumeration, it is hinted that the domain is search.htb, so I proceeded to add that to my /etc/hosts file.

```
10.10.10.129    search.htb
```

Visting search.htb in the browser returns the page shown below:
![](https://i.imgur.com/0qB9FMC.png)

I proceeded to running a directory brute force on the search.htb using gobuster and did not have much luck.

> Note: From the directory search, I found a /staff directory which returns a 403 error when trying to access it.

### There's Hope
After some time surfing through the webpage and all its deadlinks, I observed a particular image on the homepage which seems to be a diary of scheduled activities for the day.

![](https://i.imgur.com/tYb2Vcw.png)

Looks like possible name and password.

> Name: Hope Sharp
> Password: IsolationIsKey?

Since there is an SMB service running on the machine, my next line of action is to find a way to possibly enumerate SMB shares with the obtained information. One more thing, I need to determine the naming convention of the domain. For this, I generated some possible combinations like first name last name format, first name.last name format, etc. using [BridgeKeeper](https://github.com/0xZDH/BridgeKeeper).

![](https://i.imgur.com/522yWcG.png)


After generating a list, I confirmed that the username is corrected using [kerbrute](https://github.com/ropnop/kerbrute).

![](https://i.imgur.com/G0hnKNM.png)

So far, I have obtained the following information.
> Name: Hope Sharp
> Username: hope.sharp@search.htb
> Password: IsolationIsKey?

### SMB is our friend
With this credentials, I proceeded to enumerate the SMB service using [crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec).
![](https://i.imgur.com/vYydDhU.png)

This user, hope.sharp has READ access to a couple shares. I started by checking the RedirectedFolders$ share using smbclient.

```
smbclient //search.htb/RedirectedFolders$ -U hope.sharp -W search.htb
```

![](https://i.imgur.com/GkRANvM.png)

Logged into the share as this user and I created a list of users based on the directories in the share. Checked for password reuse but did not find one.

![](https://i.imgur.com/V51RdLy.png)


### What do you know bout the Three-headed dogs
Having some low-level privilege credential, I proceeded to attempt to find possible service account and obtain their password hashes for cracking using a method called [kerberoasting](https://www.qomplx.com/qomplx-knowledge-kerberoasting-attacks-explained/). For this purpose, I used the GetUserSPN.py script from [impacket](https://github.com/SecureAuthCorp/impacket).

On first run, I got an error.
```
python3 GetUserSPN.py search.htb/hope.sharp -dc-ip 10.10.11.129 -request
```

![](https://i.imgur.com/hmXiSE1.png)


> This error, KRB_AP_SKEW(clock skew too great) occured because, there is a time difference between the server clock and my local clock. If you encounter this error, you can easil fix it by synchronizing your local time with the DC time by running the command below:
> `ntpdate <DC-IP>`

After syncing with the DC clock, I ran the command again and was able to obrtain the password hash of the web_svc service account.

![](https://i.imgur.com/IEP6ngW.png)

### First step to awesomeness
I was able to crack the hash using hashcat (john works as well). Once cracked, I proceeded to check for password reuse on all other users I have obtained, and I was able to find a match.

![](https://i.imgur.com/1ke0wix.png)

The user edgar.jacobs just happens to use the same password xD.

### Again I tell you, SMB is our friend
With this new credential, I logged into the SMB share again as edgar.jacobs and obtained an Excel file named Phishing_Attempt.xlsx.

![](https://i.imgur.com/MNaPkOK.png)

![](https://i.imgur.com/nWfRtSy.png)

I downloaded the file and opened it. The file contains firstname, lastname, Username. There is also a protected column named **password**. Of course, I would love to get that.

### Unmasking the masquerade
There are a couple techniques that can be looked into when trying to recover a protected cell. One I followed can be seen [here](http://www.excelsupersite.com/how-to-remove-an-excel-spreadsheet-password-in-6-easy-steps/). In summary, the steps are:
* Make a copy of the document
* Rename the file from **Phishing_Attempt.xlsx** to **Phishing_Attempt.zip**
* Open the ZIP file using 7-zip.
* Locate the **xl** folder and then **worksheets**. Inside **worksheets**, open the **sheet2.xml** file
* Remove the **sheetProtection** tag
* Save the file
* Rename the file **Phishing_Attempt.zip** back to **Phishing_Attempt.xlsx**
* Open the file and you shoule now be able to read the password column.

![](https://i.imgur.com/c1pVgjr.png)


![](https://i.imgur.com/MNYk4hC.png)


> Note: I was able to perform this process by using a Windows machine.

### Next steps in the direction of awesomeness
I extracted all the usernames and passwords in this file and used crackmapexec to confirm the validity of the credentials. One of the username:password combinations was correct.

![](https://i.imgur.com/kY5yRzU.png)

Now we have a new user credential, sierra.frye. Again, I logged into the SMB share using this user's account.

![](https://i.imgur.com/MImE3T6.png)

This user has a Backups folder which contains certificate files. After some time researching, I found out this certificate can be imported into the browser and used by some domains. Trying to import the certificate requires a password. I used [crackpkcs12](https://github.com/crackpkcs12/crackpkcs12) to crack the password and imported the certificate successfully.

![](https://i.imgur.com/EV1lmuk.png)

### Back to basics
From initial enumeration, during directory brute forcing I found a /staff directory which returns a 403 error upon visiting. Now that I have a staff.pfx certificate imported, I tried to access the endpoint again and I was presented with a Windows Powershell Web Access console.

![](https://i.imgur.com/WGycEEk.png)

I was able to login into the domain computer using sierra.frye credentials.

![](https://i.imgur.com/KB0Std1.png)

![](https://i.imgur.com/hYPMCpk.png)


I spent quite some time here tyring to obtain a reverse shell using various technques but the AV running keeps blocking my payloads.

### Release the hounds
One of the things I tried was to download and run [bloodhound](https://github.com/BloodHoundAD/BloodHound) on the console but I was blocked. I then proceeded to use [bloodhound-python](https://github.com/fox-it/BloodHound.py) since I have some valid credentials.

> Installation: pip3 install bloodhound

```
bloodhound-python -u 'username' -p 'password' -ns DC-IP -d DOMAIN -c all
```

![](https://i.imgur.com/E8cZ1uO.png)

It ran successfully and I got the following files.

![](https://i.imgur.com/6AT0cZp.png)

Using Bloodhound GUI, I imported the files and analyzed them.


### Finding the PATH
The user sierra.frye is a member of the ITSEC@search.htb group. This group also has some permissions on the BIR-ADFS-GMSA@search.htb user, which is a [Group Managed Service Account](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview). This account also has some permission on tristan.davies user, which is a member of the Domain Admins group. So, the path looks like this:
> sierra.frye -> ITSEC@search.htb -> BIR-ADFS-GMSA@search.htb -> tristan.davies -> Domain Admin

### ReadGMSAPassword
Since sierra.frye is a member of the ITSEC@search.htb group, this user has access to all the group permissions. 


So I proceeded to reading the GMSA password of the BIR-ADFS-GMSA account using [gMSADumper](https://github.com/micahvandeusen/gMSADumper).

```
python3 gMSADumper.py -d DOMAIN -u 'username' -p 'password' 
```

![](https://i.imgur.com/FEFAs21.png)

Cool, I now have the GMSA hash.

### GenericAll
The GenericAll permission gives an account the permission to control various accounts on another account, even as far as resetting password.

![](https://i.imgur.com/RNdiPDw.png)

Snce this user has GenericAll privileges on tristan.davies, not knowing tristan's password, I proceed to [reset his password using rpcclient](https://malicious.link/post/2017/reset-ad-user-password-with-linux/) using the bir-adfs-gmsa account and the obtained hash.


```
kali@kali:~$ rpcclient -U bir-adfs-gmsa -W search.htb search.htb --pw-nt-hash
rpcclient $> setuserinfo2 tristan.davies 23 'password'
```

Now tristan's password is reset to password.

### Endgame
Since tristan is a member of the domain admin, I was now able to dump all user hashes on the machine using secretsdump.py and also get a mini-shell using smbexec.py and was able to read the root.txt file.

> I was not able to obtain a full reverse shell. If you are reading this and you were able to obtain one, please I would love to discuss the methods you used. You can reach me at admin@rudefish.wtf.

Yeah, that was a very long one. THE END!