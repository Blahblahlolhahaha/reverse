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

## Understanding Function Call Conventions
* Function calls can appear differently in assembly code
  * Governed by calling conventions
    * Order parameters are placed on the stack or in registers
    * Whether caller/function called is responsible for cleaning up the stack when function is complete
    * Depends on the compiler
    * Some conventions has to be followed when using Windows API
    * 3 main calling conventions
      * cdecl
      * stdcal
      * fastcall
  * Pseudocode:
    * ```
      int test(int x, int y, int z);
      int a, b, c, ret;

      ret = test(a,b,c);
      ```
  * ### cdecl
    * ```
      push c
      push b
      push a
      call test
      add esp, 12 (1)
      mov ret, eax
      ```
    * parameters are pushed onto the stack from right to left
    * caller cleans up the stack after function finish execution at 1
    * return value stored in EAX
  * ### stdcall
    * ```
      push c
      push b
      push a
      call test
      mov ret, eax
      ```
    * similar to cdecl
      * but function has to clean up stack by itself
        * function will be compiled differently
    * standard calling convention for Windows API
      * responsibility of DLLs to implement the code for cleaning up the stack
  * ### fastcall
    * first few arguments are being passed in registers
      * EDX and ECX commonly used
    * Additional arguments pushed from right to left
    * Function usually responsible for cleaning up the stack
    * More efficient as stack is not as involved
  * ### Push vs Move
    * Compilers can also choose to use move instead of push.
    * C code:
      * ```
        int adder(int a, int b){
          return a + b;
        }
        void main(){
          int x = 1;
          int y = 2;

          printf("the function returned the number %d\n", adder(x,y));
        }
        ```
    * Assembly:
      * adder function
        * ```
          00401730        push    ebp
          00401731        mov     ebp, esp
          00401733        mov     eax, [ebp+arg_0]
          00401736        add     eax, [ebp+arg_4]
          00401739        pop     ebp
          0040173A        retn
          ```
      * main
        * Visual Studio
          * ```
            00401746   mov     [ebp+var_4], 1
            0040174D   mov     [ebp+var_8], 2
            00401754   mov     eax, [ebp+var_8]
            00401757   push    eax
            00401758   mov     ecx, [ebp+var_4]
            0040175B   push    ecx
            0040175C   call    adder
            00401761   add     esp, 8
            00401764   push    eax
            00401765   push    offset TheFunctionRet
            0040176A   call    ds:printf
            ```
          * uses push for arguements
          * Restoration of stack pointer is needed at 0x00401761
        * GCC
          * ```
            00401085    mov     [ebp+var_4], 1
            0040108C    mov     [ebp+var_8], 2
            00401093    mov     eax, [ebp+var_8]
            00401096    mov     [esp+4], eax
            0040109A    mov     eax, [ebp+var_4]
            0040109D    mov     [esp], eax
            004010A0    call    adder
            004010A5    mov     [esp+4], eax
            004010A9    mov     [esp], offset TheFunctionRet
            004010B0    call    printf
            ```
          * Uses mov 
          * Does not need to restore stack pointer
            * It was never altered

## Analyzing Switch Statements
* Makes a decision based on a character or integer
* Compiled in 2 common ways
  * if style
  * jump tables
* If Style
  * C code:
    * ```
      switch(i){   
        case 1:      
          printf("i = %d", i+1);      
          break;   
        case 2:      
          printf("i = %d", i+2);      
          break;   
        case 3:      
          printf("i = %d", i+3);      
          break;   
        default:      
          break;
      }
      ```
  * Assembly
    * ```
      00401013        cmp     [ebp+var_8], 1
      00401017        jz      short loc_401027 (1)
      00401019        cmp     [ebp+var_8], 2
      0040101D        jz      short loc_40103D
      0040101F        cmp     [ebp+var_8], 3
      00401023        jz      short loc_401053
      00401025        jmp     short loc_401067 (2)
      00401027 loc_401027:
      00401027        mov     ecx, [ebp+var_4] (3)
      0040102A        add     ecx, 1
      0040102D        push    ecx
      0040102E        push    offset unk_40C000 ; i = %d
      00401033        call    printf
      00401038        add     esp, 8
      0040103B        jmp     short loc_401067
      0040103D loc_40103D:
      0040103D        mov     edx, [ebp+var_4] (4)
      00401040        add     edx, 2
      00401043        push    edx
      00401044        push    offset unk_40C004 ; i = %d
      00401049        call    printf
      0040104E        add     esp, 8
      00401051        jmp     short loc_401067
      00401053 loc_401053:
      00401053        mov     eax, [ebp+var_4] (5)
      00401056        add     eax, 3
      00401059        push    eax
      0040105A        push    offset unk_40C008 ; i = %d
      0040105F        call    printf
      00401064        add     esp, 8
      ```
    * Will be represented by a set of conditional jumps between 1 and 2
    * Options for the switch are at 3,4,5
      * all independant of each other due to unconditional jumps present in 3 and 4
    * Hard to differentiate between switch and if statements in assembly
      * As long as they give the same meaning it's fine
* Jump Table
  * C Code:
    * ```
      switch(i){   
        case 1:      
          printf("i = %d", i+1);      
          break;   
        case 2:      
          printf("i = %d", i+2);      
          break;   
        case 3:      
          printf("i = %d", i+3);      
          break; 
        case 4:
          printf("i = %d", i+3);      
          break; 
        default:      
          break;
      }
      ```
  * Assembly
    * ```
      00401016        sub     ecx, 1
      00401019        mov     [ebp+var_8], ecx
      0040101C        cmp     [ebp+var_8], 3
      00401020        ja      short loc_401082
      00401022        mov     edx, [ebp+var_8]
      00401025        jmp     ds:off_401088[edx*4](1)
      0040102C   loc_40102C:              
      ...
      00401040        jmp     short loc_401082
      00401042   loc_401042:              
      ...
      00401056        jmp     short loc_401082
      00401058   loc_401058:              
      ...
      0040106C        jmp     short loc_401082
      0040106E   loc_40106E:              
      ...
      00401082   loc_401082:          
      00401082        xor     eax, eax
      00401084        mov     esp, ebp
      00401086        pop     ebp
      00401087        retn
      00401087   _main   endp
      00401088 (2)off_401088 dd offset loc_40102C
      0040108C               dd offset loc_401042
      00401090               dd offset loc_401058
      00401094               dd offset loc_40106E
      ```
    * 2 defines the jump table to jump to different locations
      * switch variable will be acting as index into the jump table
        * stored in ecx in this example
        * subtracted by 1 as index starts from 0;
        * moved into edx 
        * offset is edx multiplied by 4 since each entry will be 4 bytes long and the jump happens to the jump table at 1

## Disassembling Arrays
* C code:
  * ```
    int b[5] = {1,2,3,4,5};
    void main(){
      int i;
      int a[5];
      
      for(i = 0;i<5; i++){
        a[i] = i;
        a[i] = i
      }
    }
    ```
  * a is locally defined
  * b is globally defined
    * will affect how assembly code will turn out.
 * Assembly
   * ```
     00401006        mov     [ebp+var_18], 0
     0040100D        jmp     short loc_401018
     0040100F loc_40100F:
     0040100F        mov     eax, [ebp+var_18]
     00401012        add     eax, 1
     00401015        mov     [ebp+var_18], eax
     00401018 loc_401018:
     00401018        cmp     [ebp+var_18], 5
     0040101C        jge     short loc_401037
     0040101E        mov     ecx, [ebp+var_18]
     00401021        mov     edx, [ebp+var_18]
     00401024        mov     [ebp+ecx*4+var_14], edx (1)
     00401028        mov     eax, [ebp+var_18]
     0040102B        mov     ecx, [ebp+var_18]
     0040102E        mov     dword_40A000[ecx*4], eax (2)
     00401035        jmp     short loc_40100F
     ```
    * Arrays are accessed using a base address as starting point
    * b is dword_40A000
    * a is [ebp+var_14]
    * In this example, ecx is used as index
    * ecx * 4 is because a and b are array of integers, where each element is 4 bytes long

## Identifying Structs
* C Code:
  * ```
    struct my_structure{
      int x[5];
      char y;
      double z;
    };

    struct mystructure *gms;

    void test(struct mystructure *q){
      int i;
      q->y = 'a';
      q->z = 15.6;
      for(i = 0; i<5; i++){
        q->x[i] = i;
      }
    }

    void main(){
      gms = (struct my_structure *) malloc(
        sizeof(struct my_structure)
      );
      test(gms);
    }
    ```
  * gms is a global variable
  * main allocates memory for the structure which is then passed into test
* Assembly
  * Main Function
    * ```
      00401050        push    ebp 
      00401051        mov     ebp, esp 
      00401053        push    20h             
      00401055        call    malloc 
      0040105A        add     esp, 4
      0040105D        mov     dword_40EA30, eax 
      00401062        mov     eax, dword_40EA30
      00401067        push    eax (1)
      00401068        call    sub_401000
      0040106D        add     esp, 4
      00401070        xor     eax, eax 
      00401072        pop     ebp 
      00401073        retn
      ```
  * Test Function:
    * ```
      00401000        push    ebp
      00401001        mov     ebp, esp
      00401003        push    ecx
      00401004        mov     eax,[ebp+arg_0]
      00401007        mov     byte ptr [eax+14h], 61h
      0040100B        mov     ecx, [ebp+arg_0]
      0040100E        fld     ds:dbl_40B120 (1)
      00401014        fstp    qword ptr [ecx+18h]
      00401017        mov     [ebp+var_4], 0
      0040101E        jmp     short loc_401029
      00401020 loc_401020:                             
      00401020        mov     edx,[ebp+var_4]
      00401023        add     edx, 1
      00401026        mov     [ebp+var_4], edx
      00401029 loc_401029:                             
      00401029        cmp     [ebp+var_4], 5
      0040102D        jge     short loc_40103D
      0040102F        mov     eax,[ebp+var_4]
      00401032        mov     ecx,[ebp+arg_0]
      00401035        mov     edx,[ebp+var_4]
      00401038        mov     [ecx+eax*4],edx (2)
      0040103B        jmp     short loc_401020
      0040103D loc_40103D:                           
      0040103D        mov     esp, ebp
      0040103F        pop     ebp
      00401040        retn
      ```
  * Structs are accessed with a base address used as a starting pointer
  * In main, 1 is where the base address of gms is being passed into sub_401000 which is the test function via push eax
  * In test:
    * at offset 0x18, a double is used since fld and fstp are used 
      * fld is load floating point value
      * fstp is store floating point value
      * this means 0x18 is a double
    * integers are stored into offsets 0x4,0x8,0xC,0x10 through the for loop.

## Analyzing Linked List Traversal
* 