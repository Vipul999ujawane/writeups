## License
### Category : Reversing

---
```
Description: Keith made a cool license-checking program but he forgot the flag he used to create the key! To make matters worse, he lost the source code and stripped the binary for his license-generator program. Can you help Keith recover his flag? All he knows is:

The license key is 4-EZF2M-7O5F4-V9P7O-EVFDP-E4VDO-O
He put his name (in the form of 'k3ith') as the first part of the flag
There are 3 underscores
The flag is in the format hsctf{}
The flag doesn't have random character sequences (you should be able to read the entire flag easily).
The flag only contains lowercase English letters and numbers.
The generator might produce the same keys for different inputs because Keith was too lazy to write the algorithm properly.

```

Now, I tried decompiling the binary in ghidra. After sometime of figuring out the weirdly obfuscated code, I decided to black box test it.

Started with the `A` got a `X` back. with `AA` got a  `X-X` back. Basically, seemed like some sort of substituion cipher. So. wrote a script that mapped the letters to their encrypted text. Then after reverse indexing,

I got 

```
hsctf{k}ith_m4k}s_tr4sh_r}}
```
The flag looks mostly correct, apart from a few characters. Since, we know the flag starts with a `k3ith`, Probably, the scond mapping of `0` could be `3` or `}`.

The most probable flag seems 

```
hsctf{k3ith_m4k3s_tr4sh_r3}
```
and its correct xD