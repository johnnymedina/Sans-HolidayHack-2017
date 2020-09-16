# Command Line CTF Walkthrough
The Terminal Command Line challenges are provide tools that could be used in the snowball 
challenge and in hints to complete the main CTF challenges.

## Terminal Bushy Evergreen
<p align="center"> <img src="https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/1-Terminalbushyevergreen.png" width="50%"></p>\
The challenge is to locate the 'elftalkd' binary and execute it on the system
without having 'Find' and 'Locate' avaiable to you. \

To solve just need to identify alertnative locations for 'find'.

Where is FIND?, What are known locations of FIND?
1. find/usr/bin/find /usr/local/bin/find /usr/share/man/man1/find.1.gz /usr/share/info/find.info.gz
2. moved to /usr/bin and executed './find / -name elftalkd'
3. found it at /run/elftalk/bin/elftalkd
4. moved to directory and executed './elftalkd'

![Bushy Solved](/Images/2-Terminalbushyeverygreen-solved.png)

## Terminal Winconceiveable
![Winconcievable](/Images/3-Terminalwinconceiveaable.png)\
The challenge in this terminal is that the service "kill" isnt avaiable and you have 
to find an alternative way to stop a service

1. ps aux, kill 4224 but didnt work
2. typed 'alias'
3. saw 'kill' command was aliased to 'true'
4. took hint and found http://www.linfo.org/alias.html
5. used 'unalis kill'
6. then 'kill 8'
7. ps aux showed process had been killed


## Terminal CandyCaneStriper
![Candycane](/Images/4-Terminalcandycanestriper.png)\
Challenge was to execute a script without the execution permission bit set (+x)
1. ls -ll
2. file CandyCaneStriper
3. /lib64/ld-linux-x86-64.so.2 /home/elf/CandyCaneStriper +x 


![Candysolved](/Images/5-Terminalcandycanestriper-solved.png)

## Terminal TheresSnowPlaceLikeHome
![Theres](/Images/6-Terminaltheressnowplacelikehome.png)\
Problem was to execute a program that was designed to only run on an ARM 
architecture
1. file trainstartup
2. uname -a
3. quick google 'how to run arm on x64system' led to 
found http://tuxthink.blogspot.com/2012/04/executing-arm-executable-in-x86-using.html
4. ran 'qemu-arm trainstartup'

![Theres-solved](/Images/7-Terminaltheressnowplacelikehome-solved.png)

## Terminal BumblesBounce
![Bumblebounce](/Images/8-Terminalbumblesbounce.png)\
The main challenge with this terminal was around data parsing. Given a huge log of 
HTTP requests, and picking out the most popular browser.

![Bumblebounce-info](/Images/9-Terminalbumblesbounce.png)\

1. cat access.log |grep GET | cut -d ' ' -f 12 | sort -n | uniq -c | sort -n
2. from results several had a count of  '1', but keyword in question was least popular 'browser'

![Bumblebounce-solved](/Images/10-Terminalbumblesbounce-solved.png)\

## Terminal SugarPlum
![Sugarplum](/Images/11-Terminal-Sugarplum.png)\
This challenege has data in a SQLite database and is just about crafting the correct SQL statement
to retrieve the correct data

1. Found sqlite3 running
2. .tables and found 2 tables likes, songs
3. .schema likes and .schema songs
4. Found site for reference : http://www.sqlitetutorial.net/sqlite-group-by/
5. SELECT songid, COUNT(like) FROM likes GROUP BY songid ORDER BY COUNT (like);\
    Returned: top song with likes is SONG ID:392, with 11325 likes
6. 265|2140
7. 245|2162
8. 392|11325
9. SELECT id, title FROM songs GROUP BY id;\
 Returned: 392| Stairway to Heaven

![Sugarplum-solved](/Images/12-Terminalsugarplum-solved.png)
## Terminal Shiny
![Shiny](/Images/13-Terminalshiny.png)\
Needed to repair Shinny's server access but dont have root privledges BUT do have sudo.\
1. sudo -ll
2. sudo -g shadow find /*/shadow.bak
3. man find and found it can execute a command with -exec
4. sudo -g shadow find /etc/shadow.bak -exec cp '{}' /etc/shadow \;
5. cd /usr/local/bin
6. ./inspect_da_box

![Shiny-solved](/Images/14-Terminalshinny-solved.png)

## Terminal OpenSale
![Opensale](/Images/15-Terminalopensale.png)\
The challenge for OpenSale was to make the binary you are given always return the value 42
when executed.\

1. cat isit42.c.un
2. found this reference
https://pen-testing.sans.org/blog/2017/12/06/go-to-the-head-of-the-class-ld-preload-for-the-win 
3. nano jm.c
    ```
    #include <stdio.h>
    unsigned int rand(unsigned int microseconds) 
    {printf("Hijacked rand!\n");
    return 42;}
   ```
4. gcc jm.c -o jm -shared -fPIC
LD_PRELOAD="$PWD/jm" ./isit42
COMPLETED

![Opensale-solved](/Images/16-Terminalopenslae-solved.png)

