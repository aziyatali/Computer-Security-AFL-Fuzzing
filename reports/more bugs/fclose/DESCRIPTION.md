Bug 5

Name 

Segmentation fault

Affected Lines 

In pngparcer.c Line: 704

Expected vs Observed 
If we analyze the binary file /reports/fclose/id:000048,sig:11,src:000064,op:havoc,rep:4

==13034==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x7f8b04c63f5b bp 0x000000000000 sp 0x7ffc4400a690 T0)
==13034==The signal is caused by a READ memory access.
==13034==Hint: address points to the zero page.

if we go through the code when file is opened:
  FILE *input = fopen(filename, "rb");

and then we check whether open was successfull:
  if (!input) {
    goto error;
  }

In case of failed input we go to the error message:


error:
  fclose(input);
  ....
  
The thing is we can successfully close the program only when we open it successfully, otherwise input is NULL. 
fclose takes a pointer obtained either by fopen, one of the standard streams stdin, stdout, or stderr, or in some other implementation-defined way. And NULL pointer not one of these.

SUMMARY: AddressSanitizer: SEGV (/lib/x86_64-linux-gnu/libc.so.6+0x84f5a) in _IO_fclose
==13034==ABORTING

			
Steps to Reproduce

Command: 

./size NULL 
it shows the error message like below:

==14105==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x7f8f39a88f5b bp 0x000000000000 sp 0x7ffeb1696c00 T0)
==14105==The signal is caused by a READ memory access.
==14105==Hint: address points to the zero page.
    #0 0x7f8f39a88f5a in _IO_fclose (/lib/x86_64-linux-gnu/libc.so.6+0x84f5a)
    #1 0x7f8f39d1d908 in __interceptor_fclose (/lib/x86_64-linux-gnu/libasan.so.5+0x10b908)
    #2 0x560dca354896 in load_png /home/aziyatali/Desktop/src/pngparser.c:704
    #3 0x560dca34f916 in main /home/aziyatali/Desktop/src/size.c:16
    #4 0x7f8f39a2b0b2 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x270b2)
    #5 0x560dca34fe1d in _start (/home/aziyatali/Desktop/src/size+0x2e1d)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV (/lib/x86_64-linux-gnu/libc.so.6+0x84f5a) in _IO_fclose
==14105==ABORTING

    
Proof-of-Concept Input (if needed): 
/reports/fclose/id:000048,sig:11,src:000064,op:havoc,rep:4
 

Suggested Fix Description: 

We need to check argument of fclose(file) which is file before closing it. If the case file is NULL, we need to return error message rather than letting the program crash. 
