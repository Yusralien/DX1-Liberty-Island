# DX1-Liberty-Island
Simple Writeup of rooting DX1: Liberty Island
https://tryhackme.com/room/dx1libertyislandplde

Nmap scan :
- nmap -sC -sV -v 10.10.116.49

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

Here we got 80 open which will redirect to a website, 5901 and 7778..lets enumerate

http://10.10.116.49/robots.txt
# Disallow: /datacubes # why just block this? no corp should crawl our stuff - alex

http:10.10.116.49/datacubes/0000

All credentials within *should* be [redacted] - alert the administrators immediately if any are found that are 'clear text'

Access granted to personnel with clearance of Domination/5F or higher only.

> this mean we can fuzz the directory 0000 to any numbers higher such as 0001, 0002, 0003 and so on.

crunch 4 4 0123456789 > fuzz.txt

fyi, crunch is used to generate custom keywords based on wordlists.

then i'll enumerate the directory by using dirsearch with txt extension and supported with fuzz.txt wordlists


dirsearch -u http://10.10.116.49/datacubes -e .txt -w fuzz.txt

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


we got many directories tht providing plain text which might be useful for us to exploit further.

after a deep exploring on the dir, i found something tht can lead me further which the phrase stated :>

"The VNC login is the following message, 'smashthestate', hmac'ed with my username from the 'bad actors' list (lol).
Use md5 for the hmac hashing algo. The first 8 characters of the final hash is the VNC password. - JL"

then i add badactors.html as dir and found page tht list many usernames (could be real badactor hehe) and based on the hint given..the person tht quoted tht is JL , so i assume the badactors is jlebedev

im cluelesss, then i decide to search on google what is actually hmac and vnc login, i found out tht hmac can be used as decoding, so im try to decode the text smashthestate by md5 hash and insert jlebedev as a key ..here i found the password

311781a1
the first 8 characters.

lets enumerate 5901 port tht will redirect to vnc login..

after insert the password , u will redirect to vnc and u will found the user.txt

then i spawn reverse shell 

after a few minutes exploring, i think maybe badactor list can be something tht a function for us to root the machine..then i spawn httpserver at vnc terminal to me download the badactor list from my terminal...

then i chmod the file with "chmod 777 badactors-list"

but it cant get me into the vnc, i decide to add the UNATCO in my /etc/hosts 
and intercept the traffic by using wireshark "tun0 interface"

and yeap..i found the key which 

Clearance-Code: 7gFfT74scCgzMqW4EQbu
directive=

Then, open the burpsuite 
turn on the intercept from proxy option

and intercept the web 

http://ip:23023

then send to the repeater
send to the response and dont forget to change request method
then insert the key found 

after that..try check if u are currently in root..

then by using the command

cat /root/root.txt

u will find the flag..
