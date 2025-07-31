The challenge starts with this menu:
<br>
              Menu:
              1. Flag
              2. Command prompt
              Choose an option:
<br>
Upon inspection of the source code, the following function can be found:
<br>
<img width="338" height="150" alt="image" src="https://github.com/user-attachments/assets/bc2c8c27-ab9d-4e2d-bd76-3dc12e30f2ba" />
<br>
This means that an error handler is set up such that when there is a segmentation error, the program will output the flag.
Additionally, the following function can be seen in the source code:
<br>
<img width="282" height="128" alt="image" src="https://github.com/user-attachments/assets/ff944cdf-d385-46de-841a-f5c0d3eb834e" />
<br>
Because there is a limited size in the buffer, only 16, we can create a segmentation error by creating a buffer overflow. Logically, the buffer will overflow after inputting 17 characters.
<img width="915" height="470" alt="image" src="https://github.com/user-attachments/assets/b41d7a1d-6751-4616-88b1-1ebcea20cafd" />
<br>
It can be seen above that the exploitable function vuln() can be called by choosing option 2
<br>
<img width="234" height="106" alt="image" src="https://github.com/user-attachments/assets/b6e09507-d32c-4b77-b6e1-f9d602e91ff0" />
<br>
