# ARE YOU FAST IN MATH – 150 pts
Category: Problem Solving

Solver: Cozy

Objective:
Beat a timed arithmetic quiz with nc to print the flag.

## Challenge Overview
We're given a problem from the server and expects the correct answer in less than 3 seconds.Each math problem looks like.

```python
A op B = ?
```

My first approach was to solve it fast using a calculator. But it after knowing that it is not only a few items. I quickly came up with the idea to just use a script to answer them for me.

So I searched the web for ctfs similar to this one and found a script that solves it for me. However, one of the problems I've encountered was parsing the expresion.

I had to remove the first line which was

"Are you fast in math? Let's see!"

My mind quickly realized that I can use regex for this. So I have searched the web and formed this regex to find the expression and  then use the eval function to solve it.

```python
match = re.search(r'(\d+)\s*([+\-*/^|%])\s*(\d+)', decoded)
```

So now the script is finally complete after encountering problems with the operations "^", "%", and lastly "|".

```python
import socket
import re
import operator
 
 
MAXBUF = 4096
SENTINEL = 'CITU'
CTF_BOT = ('192.168.8.136', 10010)
 
if __name__ == '__main__':
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect(CTF_BOT)
 
    while True:
        data = b''
 
        # receive and store data
        while True:
            chunk = client.recv(MAXBUF)
            data += chunk
            if len(chunk) < MAXBUF:
                break
       
        # store decoded data for future usage
        decoded = data.decode('utf-8')
       
        #temporary
      
        #
 
        # our flag likely contains flag{}, once it's revealed print received data and exit
        if SENTINEL in decoded:
            print(decoded)
            break
       
        match = re.search(r'(\d+)\s*([+\-*/^|%])\s*(\d+)', decoded)
 
        if not match:
            print(decoded)
            print(decoded.split(),"bruh")
            raise ValueError("Invalid expression string")
       
        expression = match.group(0)
 
        # properly handle division
        if '/' in expression:
            expression = expression.replace('/', '//')
        print(expression)
        if "^" in expression:
            lis = expression.split(" ")
            print(lis)
           
            try:
               result = pow(int(lis[0]),int(lis[2]))
               print(result,"this naswer")
               print(int(lis[0]),int(lis[2]))
            except ValueError:
                print(lis[2])

                result = eval(expression)
        else:
            result = eval(expression)
 
        # print results to screen to see script progress
        print(expression + ' = ' + str(result))
 
        # encode and transfer
        data = str(result).encode('utf-8') + b'\n'
        print('Sending: ' + str(result))
        client.send(data)
```
After running this solver, the flag shows up!
`CITU{0k_y0u_4r3_f4st_m4th_w1z4rd!}`
