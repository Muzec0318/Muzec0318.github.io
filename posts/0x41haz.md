---
layout: default
title : muzec - 0x41haz THM Writeup
---

Now that is a `Simple Reversing Challenge` which i find cool bypassing anti-reversing measures XD.

```
In this challenge, you are asked to solve a simple reversing solution. Download and analyze the binary to discover the password.

There may be anti-reversing measures in place!
```

![image](https://user-images.githubusercontent.com/69868171/161383607-ac21c00e-9632-4a24-8c10-b2bd715956d6.png)

Now that we have the file downloaded already let see what we are dealing with ELF 32bit lsb or ELF 64bit lsb executable.

```
┌──(muzec㉿Muzec-Security)-[~/Desktop/CTFPlayground]
└─$ file 0x41haz.0x41haz
0x41haz.0x41haz: ELF 64-bit MSB *unknown arch 0x3e00* (SYSV)
```

Now that seems like `Anti-Reverse Engineering Features` we can easily bypass it.

### MSB *unknown arch 0x3e00* (SYSV)

To bypass this, you need to patch the sixth byte (0x02) to 0x01 whic can be easily done with hexeditor.

![image](https://user-images.githubusercontent.com/69868171/161384225-3fe04ed8-c662-4d6f-abf4-38c1c458fe91.png)

Now that should be edited to `01`.

![image](https://user-images.githubusercontent.com/69868171/161384309-2aa8ef51-5164-4bb9-a4c9-04c12fde958d.png)

Now we should export it.

![image](https://user-images.githubusercontent.com/69868171/161384363-7b75f60c-79bf-4711-9f43-a63facff467e.png)

Now that is a fresh binary let confirm it, if we bypass it already.

```
┌──(muzec㉿Muzec-Security)-[~/Downloads]
└─$ file 0x41haz.0x41haz
0x41haz.0x41haz: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6c9f2e85b64d4f12b91136ffb8e4c038f1dc6dcd, for GNU/Linux 3.2.0, stripped
```

Now more better let hit `Ghidra`  to decompile it.

![image](https://user-images.githubusercontent.com/69868171/161384714-4bb96a4b-af40-42fe-9309-198d4bfe1f1f.png)

Now let go through the functions and see what we can dig out.


![image](https://user-images.githubusercontent.com/69868171/161384797-d476d1b7-b7cb-46a1-b63c-2404db6213db.png)

```

undefined8 FUN_00101165(void)

{
  size_t sVar1;
  char local_48 [42];
  undefined8 local_1e;
  undefined4 local_16;
  undefined2 local_12;
  int local_10;
  int local_c;
  
  local_1e = 0x6667243532404032;
  local_16 = 0x40265473;
  local_12 = 0x4c;
  puts("=======================\nHey , Can You Crackme ?\n=======================");
  puts("It\'s jus a simple binary \n");
  puts("Tell Me the Password :");
  gets(local_48);
  sVar1 = strlen(local_48);
  local_10 = (int)sVar1;
  if ((int)sVar1 != 0xd) {
    puts("Is it correct , I don\'t think so.");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  local_c = 0;
  while( true ) {
    if (0xc < local_c) {
      puts("Well Done !!");
      return 0;
    }
    if (*(char *)((long)&local_1e + (long)local_c) != local_48[local_c]) break;
    local_c = local_c + 1;
  }
  puts("Nope");
                    /* WARNING: Subroutine does not return */
  exit(0);
}

```

Seems like the decompile code of what we need interesting aslo we have some hex strings let decode it and reverse it.

```
local_1e = 0x6667243532404032;
local_16 = 0x40265473;
local_12 = 0x4c;


┌──(muzec㉿Muzec-Security)-[~/Downloads]
└─$ echo 0x6667243532404032 | xxd -r -p               
fg$52@@2                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Downloads]
└─$ echo 0x40265473 | xxd -r -p
@&Ts                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Downloads]
└─$ echo 0x4c | xxd -r -p      
L                                                                                                                                                                       
┌──(muzec㉿Muzec-Security)-[~/Downloads]
└─$ 
```

Inteesting we have all the piece let reverse it and put it all together. And we should have 

```
2@@25$gfsT&@L
```

Now that look better let confirm if we have the right password.

```
┌──(muzec㉿Muzec-Security)-[~/Downloads]
└─$ ./0x41haz.0x41haz
=======================
Hey , Can You Crackme ?
=======================
It's jus a simple binary 

Tell Me the Password :
2@@25$gfsT&@L
Well Done !!
```

Boom right and we are done.

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
