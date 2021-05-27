Bug 6

Name 

Heap Overflow 

Description 

Heap Overflow, we give img->px large input value than it can fit into them (out of bound access) 

Affected Lines 

In pngparcer.c Line: 464

Expected vs Observed 
If we analyze the binary file /reports/00/id:000045,sig:06,src:000008,op:int32,pos:20,val:+1
then we will notice that initial allocation taken place on line 441 in pngparser.c:
    #1 0x55b1f8e4d2bf in convert_rgb_alpha_to_image /home/aziyatali/Desktop/src/pngparser.c:441
here we allocated   img->px = malloc(sizeof(struct pixel) * img->size_x * img->size_y);

However heap overflow taking place in convert_rgb_alpha_to_image() function on line 464:

SUMMARY: AddressSanitizer: heap-buffer-overflow /home/aziyatali/Desktop/src/pngparser.c:462 in convert_rgb_alpha_to_image

In the beginning we created the size for img-> px which is in range of img->size_x * img->size_y 

Line: 441 
img->px = malloc(sizeof(struct pixel) * img->size_x * img->size_y);

However, in lines: 464 img->px is taking 
			idy * img->size_x + idx
	where: idy = idy < height (height = img->size_y)
	where  idx = idx < width (width = img->size_x)
	
	in case of idy = height-1 && idx = width-1  then:
			idy * img->size_x + idx > img->size_x * img->size_y
	which is OUT_OF_BOUND_ACCESS

we expect the value in img->px should be in range of img->size_x * img->size_y
			

Steps to Reproduce 

Command: 

./size ./seeds/rgba.png 
it shows the error message like below:

    Indirect leak of 40000 byte(s) in 1 object(s) allocated from:
    #0 0x7fe276e6bbc8 in malloc (/lib/x86_64-linux-gnu/libasan.so.5+0x10dbc8)
    #1 0x556fe40a22bf in convert_rgb_alpha_to_image /home/aziyatali/Desktop/src/pngparser.c:441

Proof-of-Concept Input (if needed): 
/reports/heapoverflow1/id:000042,sig:11,src:000001,op:flip1,pos:18

 

Suggested Fix Description: 

We need to check each time (idy * img->size_x + idx) is less than (img->size_x * img->size_y). 
If it is equal to or larger than the range we need to break and exit the loop. 

