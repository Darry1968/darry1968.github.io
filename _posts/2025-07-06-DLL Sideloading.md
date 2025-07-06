---
categories: [Malware Analysis, DLL Sideloading Attack]
tags: [dll]     # TAG names should always be lowercase
author: Darshan
description: DLL Sideloading is a technique that enables the attacker to execute custom malicious code from within legitimate or maybe even signed windows binaries/processes.
image:
    path: /assets/img/Banners/DLL-Sideloading.jpeg
pin: false
---

## What is DLL Sideloading?

DLL side-loading is a popular technique used by threat actors to circumvent security solutions and trick the Windows Operating system into executing malicious code on the target endpoint.

### What is this DLL?

A DLL (Dynamic Link Library) is like a toolbox for programs. It holds shared tools (code) that many apps can use without copying everything. It helps apps run smoother and save space. Think of it as a recipe book all your programs borrow from instead of rewriting the same recipe!

## Understand through example

Let's create a sample binary and see how exe or any program and dll works together. First we'll create main exe file using this C code. Which will load a dll file.

```c
#include <windows.h>
#include <stdio.h>

typedef void (*RUNFUNC)();

int main()
{
    HMODULE hLib = LoadLibrary("plugin.dll");
    if (hLib == NULL)
    {
        printf("Failed to load plugin.dll\n");
        return 1;
    }

    RUNFUNC runme = (RUNFUNC)GetProcAddress(hLib, "runme");
    if (!runme)
    {
        printf("Function not found in DLL\n");
        return 1;
    }

    runme();
    FreeLibrary(hLib);
    return 0;
}
```
This program loads a DLL file called `plugin.dll` at runtime. It then looks for a function named `runme` inside that DLL. If found, it calls that function. After running, it unloads the DLL. It's like loading a plugin, using one of its features, then cleaning up.
Complie this code using this command:

```bash
gcc load_dll.c -o load_dll.exe
```

This will create load_dll.exe file. Now if you run this code execution will fail because plugin.dll is not present.
![Failed-exec](/assets/img/DLL-Sideloading/failed-execution.png)

Now we create plugin.dll using this code:
```c
#include <windows.h>
#include <stdio.h>

__declspec(dllexport) void runme()
{
    MessageBox(0, "Hello from plugin!", "DLL Called", MB_OK);
}
```

Complie this code this this command:
```bash
gcc -shared -o plugin.dll plugin.c
```

This will create plugin.dll and your directory will look like this:  
![Directory](/assets/img/DLL-Sideloading/directory.png)  

Now again run this code and see if that loads the dll or not.
![success](/assets/img/DLL-Sideloading/successful-exec.png)

Boom!! It worked as it should be.  

![happy](/assets/img/meme/Happy-cat.gif)  


## Understanding the DLL Search Order

Wait then where is DLL-Sideloading?? This is the normal behaviour. We are comming to that only before that we need to understand what happens when dll is not present in the same directory?
A quick google search for dll search order [Read here](https://medium.com/@sealteamsecs/dll-search-order-hijacking-c9c46ea9026c)

![Search-order](/assets/img/DLL-Sideloading/dll-search-order.png)  
These are the locations where it will search the dll and if not found the execution will fail.

So if we place our `plugin.dll` in `C:\Windows\System32` Folder will it load? Let's see that
![System32](/assets/img/DLL-Sideloading/system32.png)  

Now we run our program again and see if it loads that dll or not?
![Failed-exec](/assets/img/DLL-Sideloading/failed2.png)

But wait it should load that dll right? The dll file is present in `System32` Folder what just happend.  

## The "Why the F isn't loading?!" Phase
![happy](/assets/img/meme/frustrated.gif){: width="500" height="400" }

Let's open procmon to see where it searches for dll. Add these filters to shorten your search:
![Procmon-filter](/assets/img/DLL-Sideloading/procmon%20filter.png)

And then run your exe again to load the results in procmon.
![Procmon](/assets/img/DLL-Sideloading/procmon.png)

There are many more but here's the shorter version of my procmon process where it did not try to search `plugin.dll` in `System32` folder instead it did search in SysWOW64. The reason is simple the architecture issue.  

- `System32` actually holds 64-bit DLLs on 64-bit Windows.
- If you run a 32-bit EXE, it is redirected by WoW64 to: `C:\Windows\SysWOW64`  

### Check architecture using file command

If you have wsl installed using this command in this folder `C:\Windows\System32`
```bash
file plugin.dll
```

Output:
```bash
/mnt/c/Windows/System32/plugin.dll: PE32 executable (DLL) (console) Intel 80386, for MS Windows, 14 sections
```

## Final Win  

Now we know it's a 32 bit exe that's why it is getting redirected to `SysWOW64` move your plugin.dll in that folder and run it again.
And Boom!! it worked
![Success2](/assets/img/DLL-Sideloading/success.png)

Ofcourse you can place this file in any other folder also but understanding why it doesn't work in that folder only is also important.

## Key Lesseons
This shows how programs load DLLs dynamically. If a program looks for a missing DLL, you can drop a malicious one with the same name in the right spot — a common trick in DLL hijacking to evade detection and gain control.

- Don’t ignore architecture
- Procmon is your best friend
- PATH search comes last

*Thank you for reading!! ;)*