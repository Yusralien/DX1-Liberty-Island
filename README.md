# DX1-Liberty-Island
Simple Writeup of rooting DX1: Liberty Island
---
Hye, this is my first writeup doing root on vulnerable machine. Below i will paste the room's link that created by *Aquienas*                      
[DX1:Liberty Island](https://tryhackme.com/room/dx1libertyislandplde)  
	
---
# Port Scanning
First thing we gonna do if we are doing root on vulnerable machine is **scanning for the ports of victim's  machine**.

For port scanning, i have used `nmap` tools with the following command : 

```
# nmap -sC -sV -v 10.10.116.49
PORT     STATE    SERVICE   VERSION
80/tcp   open     http      Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
| http-robots.txt: 2 disallowed entries 
|_/datacubes *
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: United Nations Anti-Terrorist Coalition
5901/tcp open     vnc       VNC (protocol 3.8)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_vnc-info: ERROR: Script execution failed (use -d to debug)
7778/tcp filtered interwise
```
The scan identified ports 80, 5901, and 7778 as opens, indicating that the server has ***tcp***, ***vnc*** and ***http*** services running. We also noticed that on port 80, we are given hint which lead us to enumerate the robots.txt

# Webpage

Opening the webpage with firefox , "UNATCO" webpage is shown

<img src="https://github.com/Yusralien/DX1-Liberty-Island/blob/main/webpage.png" width="1920" height="500" /> 


In order to enumerate webpage, we have to access /robots.txt. 





