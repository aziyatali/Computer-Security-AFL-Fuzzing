Bug 3

Name 

Heap Overflow 

Description 

Heap Overflow, inflated_buf taking large input value than it can fit into them (out of bound access) 

Affected Lines 

In pngparcer.c Line: 449

Expected vs Observed 
First lets go through the error message we got:
    #1 0x55e255b991d0 in decompress_png_data /home/aziyatali/Desktop/src/pngparser.c:358
In this line we are indirectly allocating size for inflated_buf.
    output_buffer = realloc(output_buffer, output_length + have);
    *decompressed_data = output_buffer;
Later on, decompressed_data is equal to the inflated_buf as function take place in.

Heap overflow occured on line 449 when we are cheking validity of inflated_buf.
    if (inflated_buf[idy * (1 + 4 * width)])
However, we are taking index of idy * (1 + 4 * width) without knowing the size of inflated_buf can take.
Also, inflated_buf is uint8_t type of buffer size while the index we are taking is uint32_t type of width multiplied by width plus one.
In the case of large width it is highly possible that there will be heap overflow which is OUT_OF_BOUND_ACCESS

Here is ASAN summary:
SUMMARY: AddressSanitizer: heap-buffer-overflow /home/aziyatali/Desktop/src/pngparser.c:449 in convert_rgb_alpha_to_image
			

Steps to Reproduce 

Command: 

./size ./seeds/rgba.png
It shows the error message of: 

    Direct leak of 40100 byte(s) in 1 object(s) allocated from:
    #0 0x7fe276e6bffe in __interceptor_realloc (/lib/x86_64-linux-gnu/libasan.so.5+0x10dffe)
    #1 0x556fe40a11d0 in decompress_png_data /home/aziyatali/Desktop/src/pngparser.c:358
which later will be equal to inflated_buf

Proof-of-Concept Input (if needed): 
/seeds/rgba.png and attached binary file in reports/02/id:000043,sig:06,src:000001,op:flip1,pos:20


Suggested Fix Description: 

We need to first check the buffer size of inflated_buf and then limit the index we are taking according to the size of buffer.

If size is empty or not allocated then we need to allocate new one.




