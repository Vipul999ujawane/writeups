## Real Reversal
### Category : Misc

---

```
My friend gave me some fancy text, but it was reversed, and so I tried to reverse it but I think I messed it up further. Can you find out what the text says?

```

On opening the file, the file seemed garbage, so probably the bytes in the file are reversed. 

Wrote a python script to reverse it for us:
```
with open("reversed.txt","rb") as f:
    a = f.read()
    a = a[:-1]
    b = a[::-1]
    with open("rev","wb") as f2:
        f2.write(b)
```

and bam!, we get ascii (or at least readable file). But know even the ascii is reversed.

Using python again

```
with open("rev","r") as f:
    a = f.read()
    b=a[::-1]
    with open("rev2.txt","w") as f2:
        f2.write(b) 
```

On searching `{` in the file we get `𝚃𝚑𝚎 𝚏𝚕𝚊𝚐 𝚒𝚜 𝚑𝚜𝚌𝚝𝚏{𝚞𝚝𝚏𝟾_𝚏𝚘𝚛_𝚝𝚑𝚎_𝚠𝚒𝚗}, 𝚞𝚜𝚒𝚗𝚐 𝚛𝚎𝚐𝚞𝚕𝚊𝚛 𝚊𝚜𝚌𝚒𝚒 𝚕𝚎𝚝𝚝𝚎𝚛𝚜.` Copy pasting the flag didn't help, as apparently the characters in the file are not ascii. So, typed it out to get the points.

```
FLAG : hsctf{utf8_for_the_win}
```

`rev.py`
```
with open("reversed.txt","rb") as f:
    a = f.read()
    a = a[:-1]
    b = a[::-1]
    with open("rev","wb") as f2:
        f2.write(b)

with open("rev","r") as f:
    a = f.read()
    b=a[::-1]
    with open("rev2.txt","w") as f2:
        f2.write(b) 
```
PS : A small explanation of the exploit. In python3, while opening a file, if the mode is `rb`. then byte stream is read into the buffe, whereas if `r` mode is used, strings are read.