# Chapter 6: Recognizing C Code Constructs In Assembly
* When disassembling malware
  * Should not evaluate each instruction indidvidually
    * Do this if absolutely necessary
  * Should obtain a high level picture of code by analyzing instructions in groups

## Global vs Local Variables
* Both declared similar in C    
  * But look different in assembly
* Global variables
  * can be accessed and used by any function in a program 
  * variables are declared outside of function
  * referenced by memory address in assembly
  * C Code:
    * ```
      int x = 1;
      int y = 2;

      void main(){
          x = x+y;
          printf("Total = %d\n,x);
      }
      ```
  * Assembly:
    * ```
      00401003        mov     eax, dword_40CF60
      00401008        add     eax, dword_40C000
      0040100E        mov     dword_40CF60, eax 
      00401013        mov     ecx, dword_40CF60
      00401019        push    ecx
      0040101A        push    offset aTotalD  ;"total = %d\n"
      0040101F        call    printf
      ```
   


* Local variables
  *  can be accessed only by the function in which they are defined
  *  variables are defined within the function
  *  referenced by stack addresses
  *  can change dummy names to legit stuff to make analysing easier
  *  C Code: 
     * ```
       void main(){
           int x = 1;
           int y = 1;
           x = x + y;
           printf("Total = %d\n",x);
       }
       ```
  *  Assembly:
     *  ```
        00401006        mov     dword ptr [ebp-4], 0
        0040100D        mov     dword ptr [ebp-8], 1
        00401014        mov     eax, [ebp-4]
        00401017        add     eax, [ebp-8]
        0040101A        mov     [ebp-4], eax
        0040101D        mov     ecx, [ebp-4]
        00401020        push    ecx
        00401021        push    offset aTotalD  ; "total = %d\n"
        00401026        call    printf
        ```
    * Assembly with labelling:
      * ```
        00401006        mov     [ebp+var_4], 0
        0040100D        mov     [ebp+var_8], 1
        00401014        mov     eax, [ebp+var_4]
        00401017        add     eax, [ebp+var_8]
        0040101A        mov     [ebp+var_4], eax
        0040101D        mov     ecx, [ebp+var_4]
        00401020        push    ecx
        00401021        push    offset aTotalD  ; "total = %d\n"
        00401026        call    printf
        ```

## Disassembling Arithmetic Operations
* C Code:
  * ```
    int a = 0;
    int b = 0;
    a = a + 11;
    a = a - b;
    a--;
    b++;
    b = a % 3;
    ```
* Assembly:
  * ``` 
    00401006        mov     [ebp+var_4], 0
    0040100D        mov     [ebp+var_8], 1
    00401014        mov     eax, [ebp+var_4] (1)
    00401017        add     eax, 0Bh
    0040101A        mov     [ebp+var_4], eax 
    0040101D        mov     ecx, [ebp+var_4]
    00401020        sub     ecx, [ebp+var_8] (2)
    00401023        mov     [ebp+var_4], ecx 
    00401026        mov     edx, [ebp+var_4]
    00401029        sub     edx, 1 (3)
    0040102C        mov     [ebp+var_4], edx 
    0040102F        mov     eax, [ebp+var_8]
    00401032        add     eax, 1 (4)
    00401035        mov     [ebp+var_8], eax 
    00401038        mov     eax, [ebp+var_4]
    0040103B        cdq 
    0040103C        mov     ecx, 3
    00401041        idiv    ecx 
    00401043        mov     [ebp+var_8], edx (5)
    ```
  * a and b are being refernced by stack address
    * a is [ebp+var_4]
    * b is [ebp+var_8]
  * a is moved into and eax at 1 and then incremented by 0x0b or 11;
  * a is then moved into ecx and subtraced by b at 2
  * a is subsequently decremented at 3 while b is incremented at 4
    * instead of using inc or dec, compiler used add and sub
  * Last 5 is the modulo function, whereby the division instruction stores the result in eax and remainder in edx.

## Recognizing if Statements
* Used to alter program executon base on certain conditions
* Simple if Statement in 
  * C code:
    * ```
      int x = 1;
      int y = 2;
      if(x == y){
          printf("x equals to y.\n");
      }
      else{
          printf("x is not equal to y");
      }
      ```
    * Assembly
      * ```
        00401006        mov     [ebp+var_4], 1
        0040100D        mov     [ebp+var_8], 2
        00401014        mov     eax, [ebp+var_4]
        00401017        cmp     eax, [ebp+var_8] (1)
        0040101A        jnz     short loc_40102B (2)
        0040101C        push    offset aXEqualsY_ ; "x equals y.\n"
        00401021        call    printf
        00401026        add     esp, 4
        00401029        jmp     short loc_401038 (3)
        0040102B loc_40102B:
        0040102B        push    offset aXIsNotEqualToY ; "x is not equal to y.\n"
        00401030        call    printf
        ```
      * 1 is where the comparison between x and y occurs
        * var 4 is x
        * var 8 is y
      * 2 jumps when the x and y is not equal where the comparison does not return 0 else it will continue
      * 3 jumps over the else statement
    * ### Nested if Statements
      * C Code:
        * ```
          int x = 0;
          int y = 1;
          int z = 2;
          if(x == y){
              if(z == 0){
                  printf("z is zero and x = y.\n");
              }
              else{
                  "z is non-zero and x = y.\n");
              }
          }
          else {
              if(z == 0){
                  printf("z is zero and x != y.\n");
              }
              else{
                  "z is non-zero and x != y.\n");
              }              
          }
          ```
      * Assembly
        * ```
          00401006        mov     [ebp+var_4], 0
          0040100D        mov     [ebp+var_8], 1
          00401014        mov     [ebp+var_C], 2
          0040101B        mov     eax, [ebp+var_4]
          0040101E        cmp     eax, [ebp+var_8]
          00401021        jnz     short loc_401047 (1)
          00401023        cmp     [ebp+var_C], 0
          00401027        jnz     short loc_401038 (2)
          00401029        push    offset aZIsZeroAndXY_ ; "z is zero and x = y.\n"
          0040102E        call    printf
          00401033        add     esp, 4
          00401036        jmp     short loc_401045
          00401038 loc_401038:                             
          00401038        push    offset aZIsNonZeroAndX ; "z is non-zero and x = y.\n"
          0040103D        call    printf
          00401042        add     esp, 4
          00401045 loc_401045:                            
          00401045        jmp     short loc_401069
          00401047 loc_401047:                             
          00401047        cmp     [ebp+var_C], 0
          0040104B        jnz     short loc_40105C (3)
          0040104D        push    offset aZZeroAndXY_ ; "z zero and x != y.\n"
          00401052        call    printf
          00401057        add     esp, 4
          0040105A        jmp     short loc_401069
          0040105C loc_40105C:                             
          0040105C        push    offset aZNonZeroAndXY_ ; "z non-zero and x != y.\n"
          00401061        call    printf
          ```
        * 3 different jumps here
          * 1 is when x != y / var_4 != var_8
          * 2 and 3 is when var_C/z != 0;

## Recognizing Loops
* ### Finding for Loops
  * 4 components
    * initialization
    * comparison
    * execution instructions
    * increment/decrement
  * C Code
    * ```
      int i;

      for(i=0;i<100;i++>){
          printf("i equals %d\n", i);
      }
      ```
      * initialization sets i to 0;
      * i is compared to 100
        * Will execute instructions until i equals to/ is greater than 100
      * i is being incremented after each execution
  * Assembly
    * ```
      00401004        mov     [ebp+var_4], 0 (1)
      0040100B        jmp     short loc_401016 (2)
      0040100D loc_40100D:
      0040100D        mov     eax, [ebp+var_4] (3)
      00401010        add     eax, 1
      00401013        mov     [ebp+var_4], eax (4)
      00401016 loc_401016:
      00401016        cmp     [ebp+var_4], 64h (5)
      0040101A        jge     short loc_40102F (6)
      0040101C        mov     ecx, [ebp+var_4]
      0040101F        push    ecx
      00401020        push    offset aID  ; "i equals %d\n"
      00401025        call    printf
      0040102A        add     esp, 8
      0040102D        jmp     short loc_40100D (7)
      ```
    * 1 is the initialization where i is set to 0
    * 2 jumps to 5 where the comparison to 100 is made which is shown as 0x64
    * if greater, it will jump out of the loop at 6 
    * else it will execute the insturction until it reaches 7 where it jumps back to 3
    * 3 and 4 will be incrementing i by 1 using the add instruction
  * ## Finding while loops
    * Easier to understand
    * C Code:
      * ```
        int status = 0;
        int result = 0;

        while(status == 0){
            result = performAction();
            status = checkResult(result);
        }
        ```
    * Assembly
      * ```
        00401036        mov     [ebp+var_4], 0
        0040103D        mov     [ebp+var_8], 0
        00401044 loc_401044:
        00401044        cmp     [ebp+var_4], 0
        00401048        jnz     short loc_401063 (1)
        0040104A        call    performAction
        0040104F        mov     [ebp+var_8], eax
        00401052        mov     eax, [ebp+var_8]
        00401055        push    eax
        00401056        call    checkResult
        0040105B        add     esp, 4
        0040105E        mov     [ebp+var_4], eax
        00401061        jmp     short loc_401044 (2)
        ```
      * at 1, a conditional jump will occur if status, or var_4 is 0.
      * at 2, a unconditional jump will occur to go through the comparison again and again until the conditional jump occurs