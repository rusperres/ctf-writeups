The challenge starts with this in the terminal:<br>
<img width="739" height="35" alt="image" src="https://github.com/user-attachments/assets/efada306-99e3-46da-93f8-6e34f35eb764" />
<br>
The logic below can be found on the main function of the source code:<br>
<img width="581" height="333" alt="image" src="https://github.com/user-attachments/assets/42ea0b92-c4cf-4fbb-99de-e06df2a6d467" />
<br>
Taking into account that this is a c source code, accessing array elements do not undergo bounds-checking. Meaning that even though the buffer is only 64 in size, 
it is possible to create a string beyond 64 characters. The flag will be outputted as long as r', 'e' and 't', are in indeces 64, 65, and 66, respectively. This can be done by 
inputting random characters for the first 64 bytes and then input 'r', 'e' and 't'.<br>
<img width="726" height="71" alt="image" src="https://github.com/user-attachments/assets/0b3f8cb9-42c2-4b2a-aeed-05a8ffb960af" />
