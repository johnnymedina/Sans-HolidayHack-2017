# Command Line CTF Walkthrough
The Terminal Command Line challenges are provide tools that could be used in the snowball 
challenge and in hints to complete the main CTF challenges.

<br/>

## Terminal Bushy Evergreen
The challenge is to locate the 'elftalkd' binary and execute it on the system
without having 'Find' and 'Locate' avaiable to you. \

To solve just need to identify alertnative locations for 'find'.
<p align="left"> 
<img src="https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/1-Terminalbushyevergreen.png" width="40%">
</p>


Where is FIND?, What are known locations of FIND?
1. find/usr/bin/find /usr/local/bin/find /usr/share/man/man1/find.1.gz /usr/share/info/find.info.gz
2. moved to /usr/bin and executed './find / -name elftalkd'
3. found it at /run/elftalk/bin/elftalkd
4. moved to directory and executed './elftalkd'

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/2-Terminalbushyeverygreen-solved.png" width="40%">
</p>
<br/>

## Terminal Winconceiveable
The challenge in this terminal is that the service "kill" isnt avaiable and you have 
to find an alternative way to stop a service

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/3-Terminalwinconceiveaable.png" width="50%">
</p>

1. ps aux, kill 4224 but didnt work
2. typed 'alias'
3. saw 'kill' command was aliased to 'true'
4. took hint and found http://www.linfo.org/alias.html
5. used 'unalis kill'
6. then 'kill 8'
7. ps aux showed process had been killed
<br/>

## Terminal CandyCaneStriper
Challenge was to execute a script without the execution permission bit set (+x)
<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/4-Terminalcandycanestriper.png" width="50%">
</p>

1. ls -ll
2. file CandyCaneStriper
3. /lib64/ld-linux-x86-64.so.2 /home/elf/CandyCaneStriper +x 


<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/5-Terminalcandycanestriper-solved.png" width="50%">
</p>
<br/>

## Terminal TheresSnowPlaceLikeHome
Problem was to execute a program that was designed to only run on an ARM 
architecture

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/6-Terminaltheressnowplacelikehome.png" width="50%">
</p>


1. file trainstartup
2. uname -a
3. quick google 'how to run arm on x64system' led to 
found http://tuxthink.blogspot.com/2012/04/executing-arm-executable-in-x86-using.html
4. ran 'qemu-arm trainstartup'

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/7-Terminaltheressnowplacelikehome-solved.png" width="50%">
</p>
<br/>

## Terminal BumblesBounce
The main challenge with this terminal was around data parsing. Given a huge log of 
HTTP requests, and picking out the most popular browser.

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/9-Terminalbumblesbounce.png" width="50%">
</p>


1. cat access.log |grep GET | cut -d ' ' -f 12 | sort -n | uniq -c | sort -n
2. from results several had a count of  '1', but keyword in question was least popular 'browser'

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/10-Terminalbumblesbounce-solved.png" width="50%">
</p>
<br/>

## Terminal SugarPlum
This challenege has data in a SQLite database and is just about crafting the correct SQL statement
to retrieve the correct data
<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/11-Terminal-Sugarplum.png" width="50%">
</p>

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

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/12-Terminalsugarplum-solved.png" width="50%">
</p>
<br/>

## Terminal Shiny
Needed to repair Shinny's server access but dont have root privledges BUT do have sudo.\
<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/13-Terminalshiny.png" width="50%">
</p>

1. sudo -ll
2. sudo -g shadow find /*/shadow.bak
3. man find and found it can execute a command with -exec
4. sudo -g shadow find /etc/shadow.bak -exec cp '{}' /etc/shadow \;
5. cd /usr/local/bin
6. ./inspect_da_box

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/14-Terminalshinny-solved.png" width="50%">
</p>

<br/>

## Terminal OpenSale
The challenge for OpenSale was to make the binary you are given always return the value 42
when executed.

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/15-Terminalopensale.png" width="50%">
</p>

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

<p align="left"> 
<img src= "https://github.com/johnnymedina/Sans-HolidayHack-2017/blob/master/Images/16-Terminalopenslae-solved.png" width="50%">
</p>

