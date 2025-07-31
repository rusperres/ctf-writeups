# CTF Writeup: Fibonacci Encryption

Solver: Dalrho

## Challenge Overview

This cryptography challenge gave a Python script and a hash:

```
3390560246d23217f6e5f1d5295c3f2bed83a8f4a76891a8ce3b8f4ced15e0dc5bf4c26b820b98eda80c3afd5790ba11
```

The goal was to reverse the process in the script and recover the original flag.

---

## The Original Code

```python
import sys
from hashlib import sha512
from os import urandom

FLAG = b"CITU{????????????????????????????}"

def fib(k):
    if k == 0:
        return 0
    if k == 1:
        return 1
    return fib(k - 1) + fib(k - 2)

def gen_key(k):
    n = fib(k)
    h = sha512(str(n).encode()).digest()
    return h

def pad(m):
    return m + b"".join([urandom(1) for _ in range(16 - (len(m) % 16))])

def unpad(m):
    return m[:len(FLAG)]

def encrypt(m, key):
    m = pad(m)
    c = bytes([a ^ b for a, b in zip(m, key)])
    return c

def decrypt(c, key):
    m = bytes([a ^ b for a, b in zip(c, key)])
    m = unpad(m)
    return m

k = 0xC0FFEE
key = gen_key(k)
ct = encrypt(FLAG, key)
with open("output.txt", "w") as f:
    f.write(ct.hex())

pt = decrypt(ct, key)
assert pt == FLAG
```

---

## Exploitation Strategy

The hash provided is a SHA3-384 digest, which is a one-way function. That means we cannot reverse it directly — we need to work through the code to derive the key.

The key was generated from:
- `fib(k)`: the k-th Fibonacci number
- Converted to a string, then hashed with `sha512`

However, the original `fib(k)` is implemented recursively, which is extremely inefficient for large `k`. In this case, `k = 0xC0FFEE`, which is over 12 million — this would crash with recursion depth errors or take forever.

So I replaced the recursive method with a **fast-doubling Fibonacci algorithm**, which computes `fib(k)` in `O(log k)` time.

Once `fib(k)` was computed, I hashed it with `sha512` to get the encryption key, and XOR'ed that against the ciphertext.

I also had to add:

```python
sys.set_int_max_str_digits(10_000_000)
```

to let Python convert extremely large integers to strings (needed for `sha512(str(n).encode())`).

---

## My Decryption Code

```python
import sys
from hashlib import sha512

sys.set_int_max_str_digits(10_000_000)

def fib_pair(n):
    if n == 0:
        return (0, 1)
    a, b = fib_pair(n >> 1)
    c = a * ((b << 1) - a)
    d = a * a + b * b
    return (d, c + d) if n & 1 else (c, d)

def fib(k):
    return fib_pair(k)[0]

def gen_key(k):
    n = fib(k)
    return sha512(str(n).encode()).digest()

def unpad(m):
    return m[:33]  # known flag length

def decrypt(c, key):
    m = bytes([a ^ b for a, b in zip(c, key)])
    return unpad(m)

if __name__ == "__main__":
    ct_hex = "3390560246d23217f6e5f1d5295c3f2bed83a8f4a76891a8ce3b8f4ced15e0dc5bf4c26b820b98eda80c3afd5790ba11"
    ct = bytes.fromhex(ct_hex)

    k = 0xC0FFEE
    key = gen_key(k)
    pt = decrypt(ct, key)
    print("Decrypted flag:", pt.decode())
```

---

## Final Result

The decrypted flag is:

```
CITU{Fasteist_Fib0Fib0_encrpyti0n}
```
