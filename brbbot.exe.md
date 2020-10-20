## Reversing BRBBOT malware

lessons learned:
For Windows x64 binaries, setting a breakpoint on a function call, it's possible to derive the arguments being passed using the "fastcall" convention.  
The convention states registers should be used for the first 4 arguments, and the stack frame for subsequent arguments
[details from Microsoft](https://docs.microsoft.com/en-us/cpp/build/x64-software-conventions?view=vs-2019)

in x64dbg, this is represented as follows:
![](fastcall-1.png)

1. The line at which execution is paused by our breakpoint. We can see [CryptDecrypt](https://docs.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptdecrypt) is being called
   called at this step.

2. Here we see the arguments being passed to CryptDecrypt, each argument in it's own register or stack frame.
   Per the CryptDecrypt documentation, the fifth parameter (at RSP+20) points to the location in memory with the data to be decrypted.

3. This is the encrypted data at that memory location stored in RSP+20
