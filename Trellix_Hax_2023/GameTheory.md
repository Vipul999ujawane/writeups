## Classic Game Theory
### Category: Reverse Engineering, Crypto

So, in this challenge we are given a 16-BIT DOS Executable.

To run it, we can use DOSBox, with Debugging enabled.

[DOSBox](https://www.dosbox.com/)

[DosBox Debugging](https://www.vogons.org/viewtopic.php?t=3944)

----

A quick description of how the binary runs based on the disassembly.

1. It has a 672 chars of already calculated string to which our input is compared. All the chars are of the form [0-9aA-F] , so probably hex encoded bytes.

2. Our input is hashed.

3. The hash works as follows :
    
    a. Two characters are taken from the input string.
    b. The two characters are passed to this `"hasher()"` (self renamed) function.
    c. The hasher function returns a 16-byte value, which is compared to precalculated values.
    d. Once the comparison is done, the next two characters are taken from the input string and the process is repeated.
    e. We need to get all characters correct.

Things we know :

1. The flag starts with `"ARC{"`

2. We see `hasher("AR")` maps to `"ca6e65703b22403198ffff22bc32e1ca"`

3. On disassembling `"hasher()"` I found the two magic numbers `0x67452301` and `0xefcdab89`

4. These values resemble the MD5 hashing function. But, `MD5("AR")` == `5b61a1b298a0d06efa6933a97e68d763`

So, after further reverse engineering,

MD5 has 4 words in it's state. 

In standard MD5, the state is initialized to the following values :

```
A = 0x67452301
B = 0xefcdab89
C = 0x98badcfe
D = 0x10325476
```

But, in our case, the state is initialized to the following values :

```
A = 0x67452301
B = 0xefcdab89
C = 0xabcdef00
D = 0x12345678
```

I used this [MD5 library](https://github.com/Utkarsh87/md5-hashing/blob/master/md5.py) and modified the initial state as required to get the hash.

Next is a simple rainbow table attack, with rainbow tables generated using 2 character long strings.

`input` has all the precalculated hashes from the binary.

```python
from md5 import md5

input = []
with open("input") as f:
    input = f.read().split('\n')

vals = ""
for line in input:
    if line:
        vals+=(chr(int(line.split('=')[-1][:-1],16)))

#print(vals)
vals = vals.lower()
hashes = []
for i in range(0,len(vals),32):
    hashes.append(vals[i:i+32])

rainbow_table = {}

for i in range(0x00,0x7f):
    for j in range(0x00,0x7f):
        rainbow_table[md5(chr(i)+chr(j))] = chr(i)+chr(j)

for i in hashes:
    try:
        print(rainbow_table[i],end='')
    except:
        continue
```

Output :

```
┌──(greenpanda999㉿zacian)-[~/Desktop/Hax/game]
└─$ python3 solve.py 
ARC{y0u_g0tt4_pl4y_the_l0ng_g4me}                                                         
```