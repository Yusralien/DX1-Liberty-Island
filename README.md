# DX1-Liberty-Island
Simple Writeup of rooting DX1: Liberty Island
---
Hye, this is my first writeup doing root on vulnerable machine. Below i will paste the room's link that created by *Aquienas*                      
[DX1:Liberty Island](https://tryhackme.com/room/dx1libertyislandplde)  
	
---
# Port Scanning
First thing we gonna do if we are doing root on vulnerable machine is **scanning for the open ports of victim's  machine**.

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

![0451](https://user-images.githubusercontent.com/87742813/197827673-eae6ebe9-2171-4e9f-9b39-ba395fde9774.png)

Then i add badactors.html as a dir and found page tht list many usernames (could be real badactor hehe) and based on the hint given, the person that quoted is JL , so i assume the badactors is jlebedev.

![badactors](https://user-images.githubusercontent.com/87742813/197828856-ddc33a64-180e-4de9-8f88-b127954987ff.png)

At this point, im so clueless then i decide to search on google what is actually **hmac** and **vnc login**, i found out tht hmac can be used as decode type, so i tried to decode the clue `smashthestate` by md5 hashing and insert `jlebedev` as a key. 

![cybershef](https://user-images.githubusercontent.com/87742813/197831711-ed3deaa5-e5ba-48c0-9d76-00169d3c2508.png)

and we will get the password as mentioned that the first 8 character of decoded text which is **311781a1**.

---
# Initial Foothold

I started the exploitation by connecting on **VNC-login** by using the command :

`vncviewer ip_addr:5901` and followed by inserting the password
 
 ![vnc-login](https://user-images.githubusercontent.com/87742813/197833417-c90cad1e-f1aa-4e6d-bfce-84b432214354.png)

#### 1ST flag achieved

Here i got the user.txt 

![user txt](https://user-images.githubusercontent.com/87742813/197834198-38560d4d-c3a4-43e0-9a4c-31c73bcfc8a3.png)

---
# Root

After a few minutes researching and exploring, i think maybe badactor list can be something like a function for us to root the machine. So, i spawn the command :

`python3 -m http.server 8080` on the vnc-terminal and following by `wget http://ip_addr/badactors-list` to download the badactor file to my attack machine.

![wget](https://user-images.githubusercontent.com/87742813/197835569-871d973c-d20e-4728-b2bc-687d7639e1ee.png)

then i change the permission of badactors-list file by using the command :
`chmod 777 badactors-list` and continued by `./badactors-list`

![badactor](https://user-images.githubusercontent.com/87742813/197836686-401f6dbf-322d-4af5-bc76-dc7378018f19.png)

but it cant get me into the vnc, i decide to add the UNATCO in my /etc/hosts and run it again

![etc](https://user-images.githubusercontent.com/87742813/197837478-afd476cf-6b8a-4754-be63-203fee35640a.png)

then i intercept the traffic by using **Wireshark** with *tun0* interface.

![wireshark](https://user-images.githubusercontent.com/87742813/197837732-f86b4714-286b-4221-989a-44ba71c3484d.png)

Boom!! we got a few hints by following the **TCP stream** that maybe useful for us to root the machine.

* Port: 23023
* Clearance-Code: 7gFfT74scCgzMqW4EQbu
* directive:cat+%2Fvar%2Fwww%2Fhtml%2Fbadactors.txt

---
Then i open my Burpsuite, open the proxy and turn on the intercept in order to catch the raw traffic on the host `http://ip_addr:23023` 

Send the traffic caught to the repeater >> send to the response

By the way, dont forget to change request method and insert all the hints given.

![burpdirective](https://user-images.githubusercontent.com/87742813/197840435-d7d3c7b9-28ba-4458-8926-83d251fd5713.png)

After we complete the process asked, we replace:

`directive:cat+%2Fvar%2Fwww%2Fhtml%2Fbadactors.txt` to `directive:id`

![root](https://user-images.githubusercontent.com/87742813/197841105-0a0943b3-929e-45c1-80d4-639c47503796.png)

BOOM 2.0!! we got root access.

You can get the root.txt by replace `id` to `cat /root/root.txt`

Thats all from me, 

TBH, im doing this room about 3 hours(describing how noob i am...k bye





