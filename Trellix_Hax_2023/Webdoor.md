## Spying Through the Webdoor
### Category: Reverse Engineering

We are given an exectubale. Based on a quick reverse engineering, we can see that the binary is a web server.

On running `strings` through the binary, we can see an interesting one.

```
┌──(greenpanda999㉿zacian)-[~/Desktop/Hax/webdoor]
└─$ strings arc-httpd | grep ARC
Congrats! The key is: ARC%s
```

We find the xref to this string in Ghidra. To get to the string, we can see the following function condition.

```
  if ((int)sVar2 == 0x17) {
    __s2 = (void *)do_hash(__s,0x17,&DAT_001050b0,8);
    __dest = malloc(0x17);
    memcpy(__dest,&DAT_00105ea1,0x17);
    iVar1 = memcmp(__dest,__s2,0x17);
    if (iVar1 == 0) {
      fwrite("HTTP/1.1 200 OK\r\n",1,0x11,param_1);
      fwrite("Server: arc-httpd\r\n",1,0x13,param_1);
      fwrite("Connection: close\r\n",1,0x13,param_1);
      fwrite(&DAT_00105ef3,1,2,param_1);
      fprintf(param_1,"Congrats! The key is: ARC%s\r\n",__s);
      fprintf(param_1,"\nexecuting command: %s\n",local_10c8);
      ....
```

We need to clear ```memcmp==0``, which is basically an hash match. It needs to be compared to &DAT_00105ea1

```
void * do_hash(long param_1,int param_2,long param_3,int param_4)

{
  void *pvVar1;
  int local_14;
  
  pvVar1 = malloc((long)param_2);
  if (pvVar1 == (void *)0x0) {
    perror("malloc");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  for (local_14 = 0; local_14 < param_2; local_14 = local_14 + 1) {
    *(byte *)((long)pvVar1 + (long)local_14) =
         (*(byte *)(param_1 + local_14) ^ *(byte *)(param_3 + local_14 % param_4)) + 0x42;
  }
  return pvVar1;
}

```

I wrote a quick z3 script to solve this.

```
from z3 import *

final = [ 0x41, 0xfc, 0x1b, 0x18, 0xff, 0x10, 0x0d, 0xb4, 0x1d, 0xfc, 0x1b, 0x18, 0xff, 0x11, 0x0d, 0xb7, 0x18, 0xff, 0x16, 0x0d, 0xe5, 0x04, 0x3b, 0x00 ]
param_3 = [ 0x84, 0xe2, 0x96, 0x84, 0xe2, 0x96, 0x84, 0x20, ]
param_3_bitvec = [ BitVec('param_3_%d' % i, 8) for i in range(len(param_3)) ]


def main():
    s = Solver()
    param1 = [ BitVec('param1_%d' % i, 8) for i in range(0x17) ]
    for i in range(0x17):
        s.add(param1[i] >= 0x20)
        s.add(param1[i] <= 0x7e)
        s.add(final[i] == ((param1[i] ^ param_3_bitvec[i%8])+ 0x42)%0x100)
    for i in range(8):
        s.add(param_3_bitvec[i] == param_3[i])

    print(s.check())
    vals = s.model()
    for i in range(0x17):
        print(chr(vals[param1[i]].as_long()), end='')

if __name__ == '__main__':
    main()
```

We get the flag.

```
ARCXOR_XOR_XOR_YOUR_BOAT}                                                                                      ```                                                 