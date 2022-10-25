# DX1-Liberty-Island
Simple Writeup of rooting DX1: Liberty Island
---
Hye, this is my first writeup doing root on vulnerable machine. Below i will paste the room's link that created by *Aquienas*                      
[DX1:Liberty Island](https://tryhackme.com/room/dx1libertyislandplde)  
	
---
# Port Scanning
First thing we gonna do if we are doing root on vulnerable machine is **scanning for the ports of victim's  machine**.

For port scanning, i have used `nmap` tool with the following command : 

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

# Webpage Exploitation

Opening the webpage with firefox , "UNATCO" webpage is shown

![webpage](https://user-images.githubusercontent.com/87742813/197826740-79595df1-cc7a-4cf1-9c98-da30aa18c1ed.png)



In order to enumerate webpage, we have to access /robots.txt. Below is the result /robots.txt accessed.
```
http://10.10.116.49/robots.txt
# Disallow: /datacubes # why just block this? no corp should crawl our stuff - alex
```
Lets open the dir given which `/datacubes`.

![datacubes](https://user-images.githubusercontent.com/87742813/197827055-d79f8c57-e46b-40ae-a418-3e2ae800806f.png)


After deep exploring this directory , i made up with an assumption which is :

* `/0000` given is a sign that we can fuzz the dir up to 0002 or any higher values.

So ,i used to generate custom keywords/wordlists before fuzzing the directory by using `Crunch` tool with the following command :
```
crunch 4 4 0123456789 >> fuzz.txt 
```
![crunch](https://user-images.githubusercontent.com/87742813/197827245-6fe1e022-0737-4138-bbd8-28523f380ad9.png)


Then i'll fuzz the directory by using `dirsearch` tool with txt extension and supported with fuzz.txt wordlists. Below the following command :
```
# dirsearch -u http://ip_addr/datacubes -e .txt -w fuzz.txt

Extensions: .txt | HTTP method: get | Threads: 10 | Wordlist size: 10000

Error Log: /home/alien/.dirsearch/logs/errors-22-10-25_15-10-37.log

Target: http://10.10.116.49/datacubes/

[15:10:38] Starting: 
[15:10:38] 301 -  321B  - /datacubes/0000  -> http://10.10.116.49/datacubes/0000/
[15:10:39] 301 -  321B  - /datacubes/0011  ->  http://10.10.116.49/datacubes/0011/
[15:10:40] 301 -  321B  - /datacubes/0068  ->  http://10.10.116.49/datacubes/0068/
[15:10:41] 301 -  321B  - /datacubes/0103  ->  http://10.10.116.49/datacubes/0103/
[15:10:44] 301 -  321B  - /datacubes/0233  ->  http://10.10.116.49/datacubes/0233/
[15:10:49] 301 -  321B  - /datacubes/0451  ->  http://10.10.116.49/datacubes/0451/
```

We got 5 directories from the fuzzing and after exploring each directories, i found something that interest me on dir `/0451`.

<img src="https://github.com/Yusralien/DX1-Liberty-Island/blob/main/0451.png" width="1920" height="500" />








