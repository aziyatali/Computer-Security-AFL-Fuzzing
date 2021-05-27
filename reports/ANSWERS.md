Report 

Computer Security 

Aziyat Murzamidinov 

20172018 

 

 

1) Why did you need to change is_png_chunk_valid? 

After changing the is_png_chunk_valid to always return 1, we enable read_png_chunk to always return 1 since is_png_chunk_valid() always return 1 it goes to error.  

By doing so, we can read all PNG chunks; Line: 574 in pngparser.c  

for (; !read_png_chunk(input, current_chunk); 

 

  

  

2) Why did we give you exactly TWO seeds for fuzzing? 

 If I start it from the idea why do we need seeds at all? First reason is program uses seeds to generate an input for testing the program by mutating given valid image (in our case) in seeds. Why exactly two seeds are given? I don’t really know how AFL algorithm works, but after reading couples of papers on the web, my guess is that to increase the possibility of finding more unique bugs. The next questions is how it works? After mutating the given TWO seeds to generate inputs for testing, there are thousands of similar inputs. Automated seed selection picks up the best seeds in order to maximize the total number of bugs during fuzzing.  

Source: https://en.wikipedia.org/wiki/Fuzzing 

https://arxiv.org/pdf/1808.09700.pdf 

https://arxiv.org/pdf/1807.02606.pdf 

  

3) Why did you have to use afl-gcc to compile the source (and not 

e.g. ordinary gcc)? 

Since CC is the environment variable referring to the system’s C compiler, which points to(libraries, accessible etc) depend on platform.  While gcc is the driver binary for the GNU compiler collection. Afl-gcc serves as drop-in replacement for gcc or clang, letting us recompile third-party code with required run-time environment.   

  

  

4) How many crashes in total did AFL produce? How many unique 

Crashes? 

First, I run the AFL for 43 minutes and got 51 unique crashes.  

Second, I run it 7 hours and 14 minutes. It produced 58 unique crashes 

Third, I run it for 12 hours and 30 minutes. It produced only 56 crashes 

  

  

5) (1) Why are hangs counted as bugs in AFL? (2) Which type of attack can 

they be used for? 

1) Hangs are unique test cases that cause the tested program to time out. We have set (by default) time limit of 1 second (if larger than 1) before any unique test case is classified as hang.  

 2) When timeout is too long there is vulnerability for session hijacking attacks which are session-fixation, sniffing, xss etc. Once hijacked, the attacker will be able to prevent an idle time out and can hijack a security breach away.  

  

6) Which interface of libpngparser remains untested by AFL (take a 

look at pngparser.h)? 

 

 struct pixel { 

uint8_t red; 

uint8_t green; 

uint8_t blue; 

uint8_t alpha; 

}; 

 
And  

struct image { 

uint16_t size_x; 

uint16_t size_y; 

struct pixel *px; 

}; 

 
Remains untested by AFL 

 

 

  

7) In HW1 many students encountered a crash in load_png when the 

input file with the specified name didn’t exist. Can AFL find that 

bug in this setup? 

Yes, AFL is able to find the crash in load_png when the specified file name didn’t exist. I checked the size.c file where we load a png fille with load_png. It return 0 on success and 1 when it fails. When the filename in load_png() is not speicified AFl could fine the error in line 704 in pngparser.c (fclose(input), input = fopen(filename, "rb");) 

 

  

  

8) How long did you run AFL for? If you run it for twice as long, do 

you expect to find twice as many bugs? Why? 

First, I run the AFL for 43 minutes and got 51 unique crashes.  

Second, I run it 7 hours and 14 minutes. It produced 58 unique crashes 

Third, I run it for 12 hours and 30 minutes. It produced only 56 crashes 

Fourth, I run for 14 hours. It produced 56 crashes but my laptop crashed so I needed to restart. 

If I run it as twice as long then we are not necessarily expecting twice as many bugs as I already tested above. There might be a few reasons from logical point of view:  

When AFL finds vulnerability then it will save it and test it with different inputs which can easily find more bugs in the beginning. 

As the time passes number of bugs decreased and there will be less chance for finding more bugs since AFL needs to test different paths + data coverage. For each path it tests with different inputs. 
