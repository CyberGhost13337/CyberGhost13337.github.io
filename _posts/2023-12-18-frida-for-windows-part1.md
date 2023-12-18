# Exploring Windows API Hooking with Frida: A Practical Guide

## Introduction

Frida is a powerful and flexible dynamic instrumentation toolkit that enables developers and security researchers to inject JavaScript into native applications. In this blog post, we will explore the fundamentals of Frida by diving into a practical example of hooking Windows API functions. Specifically, we will focus on functions like CreateFileW and BCryptGenerateSymmetricKey, examining their arguments and showcasing how to manipulate data during runtime.

## Understanding Frida

Frida is a dynamic code instrumentation toolkit that allows you to inject JavaScript or scriptable languages into running processes. It works on various platforms, including Windows, macOS, Linux, iOS, and Android. The primary goal of Frida is to provide a dynamic and interactive way to analyze, manipulate, and control the behavior of applications.

## How to run frida

To run the Frida script for Windows API hooking, follow these steps:

### Prerequisites

1. **Install Frida:** Ensure that Frida is installed on your system. You can install Frida by using the following command:

    ```bash
    pip install frida-tools
    ```

    Additionally, you might need to install the Frida server on the target device. Refer to the [official Frida documentation](https://frida.re/docs/installation/) for installation instructions.


2. Execute the Frida script using the following command:

```bash
frida -l path/to/your/script.js -f filename.exe
```
## Finding Exported Functions

To hook a specific function, you need to find its address. In the example, we use `Module.getExportByName` to obtain the address of functions like CreateFileW and BCryptGenerateSymmetricKey. We'll discuss how this function works and why exporting functions are crucial for dynamic analysis.

```javascript
const createFileW = Module.getExportByName(null, "CreateFileW");
```

## Hexdump and Arguments

Hexdump is a utility used to display binary data in hexadecimal format. In the context of Frida, we use hexdump to inspect the content of memory at a specific address. We'll explore the hexdump function, its arguments, such as the length of the data to display, and how it helps us visualize the content of function arguments during runtime.

```javascript
console.log(hexdump(args[0], { length: 0x50 }));
```

## Hooking Functions with Addresses

Sometimes, you may need to hook a function directly by its address. We demonstrate how to obtain the base address of the main module and calculate the address of the memcmp function dynamically. This technique is useful when you want to hook functions that are not exported or when the function's address is known.

```javascript
const mainModule = Process.enumerateModules()[0];
const memcmpAddr = mainModule.base.toInt32() + 0xAA5C;
Interceptor.attach(ptr(memcmpAddr), { /* ... */ });
```

## Modifying Data during Runtime

Frida allows you to modify data in memory during runtime, providing a powerful way to alter program behavior. In the example, we intercept the memcmp function and modify its second argument using `Memory.writeByteArray`. We'll discuss the significance of this capability and when it might be useful in various scenarios.

```javascript
const newData = [0x41, 0x42, 0x43]; // Example data to be written
Memory.writeByteArray(args[1], newData);
```

## Full Code Example

Below is the complete Frida script that combines the concepts discussed above. Also full version on my gists  [frida-hook-example.js](https://gist.github.com/CyberGhost13337/44607ee84547a344abbe9f9416548aff)

```javascript
var createFileW = Module.getExportByName(null, "CreateFileW");
const PADLEN=40;
Interceptor.attach(createFileW, {
    onEnter: function(args)
    {
        /*
        
        HANDLE CreateFileW(
        [in]           LPCWSTR               lpFileName,
        [in]           DWORD                 dwDesiredAccess,
        [in]           DWORD                 dwShareMode,
        [in, optional] LPSECURITY_ATTRIBUTES lpSecurityAttributes,
        [in]           DWORD                 dwCreationDisposition,
        [in]           DWORD                 dwFlagsAndAttributes,
        [in, optional] HANDLE                hTemplateFile
        );
        */
        console.log("CreateFileW Called");
        console.log("1. Arg(lpFileName):".padEnd(PADLEN) + Memory.readUtf16String(args[0]));
        console.log("2. Arg(dwDesiredAccess):".padEnd(PADLEN) + args[1].toString());
        console.log("3. Arg(dwShareMode):".padEnd(PADLEN) + args[2].toString());
        console.log("4. Arg(lpSecurityAttributes):".padEnd(PADLEN) + args[3].toString());
        console.log("5. Arg(dwCreationDisposition):".padEnd(PADLEN) + args[4].toString());
        console.log("6. Arg(dwFlagsAndAttributes):".padEnd(PADLEN) + args[5].toString());
        console.log("7. Arg(hTemplateFile):".padEnd(PADLEN) + args[6].toString());
        console.log("------------------------------------------------------------------------------------------")
    }
});

const mainModule = Process.enumerateModules()[0];
const memcmpaddr=mainModule.base.toInt32()+0xAA5C;


Interceptor.attach(ptr(memcmpaddr), {
    onEnter:  function(args){
        //int __cdecl memcmp(const void *Buf1, const void *Buf2, size_t Size)
        console.log("memcmp Called");
        console.log("1. Arg(Buf1):".padEnd(PADLEN) + "\n"+  hexdump(args[0],{ length: 0x50}));
        
        console.log("2. Arg(Buf2):".padEnd(PADLEN) +"\n"+  hexdump(args[1],{ length: args[2].toInt32()}));
        console.log("3. Arg(Size):".padEnd(PADLEN) + args[2].toString());
        const newData = [0x41, 0x42, 0x43]; // Example data to be written
        Memory.writeByteArray(args[1], newData)
        console.log("------------------------------------------------------------------------------------------")
        
    }
});
```