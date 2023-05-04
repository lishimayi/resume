# what will you learn

## thorey

* Everything that happens before main()
* Mach-O format
* Virtual Memory basics
* How Mach-O binaries are loaded and prepard

## practical

* How to measure
* optimizing start up time

## Mach-O terminology

File types：

* *Executable*-- Main binary for Application
* *Dylib* -- Dynamic Library (aka DSO or DLL)
* *Bundle* -- Dylib that can't be linked, only *dlopen()* e.g.plug-ins
* *Image *-- An Executable , Dylib or bundle
* *FrameWork* -- Dylib with directory for resources and headers

## Mach-O Image File

### file [divided](划分) into [segments](分段)

* Uppercase names

### All the segments are multibles of page size 

* [16kb on arm64](就是arm64 处理器的一页大小是6kb)
* 4kb elsewhere

### sections are subrange of a segments

* Lowercase names

### Common segments:

* __TEXT has header . code ,and readonly constants
* __DATA has all read-write content: globals, static variables , etc
* __LINKEDIT has "Meta data " about how to load the program 

## Mach-O Universal Files <!--通用文件-->

<!--armv7： 32位-->

### Fat header

* one page size
* Lists architectures and offsets 

### Tool and runtimes support fat mach-o files

## Virtual Memory 

Virtual Memory is a level of indirection

## mach-o image loading

<!--RAM: 随机存取存储器,  随机存取存储器（英语：Random Access Memory，缩写：RAM），也叫主存，是与CPU直接交换数据的内部存储器。它可以随时读写（刷新时除外），而且速度很快，通常作为操作系统或其他正在运行中的程序的临时数据存储介质。RAM工作时可以随时从任何一个指定的地址写入（存入）或读出（取出）信息。它与ROM的最大区别是数据的易失性，即一旦断电所存储的数据将随之丢失。RAM在计算机和数字系统中用来暂时存储程序、数据和中间结果-->
you can reclaim them when someone else needs RAM. So the result is now we have two processes sharing the dylibs, each one would have been eight pages or a total of 16 dirty pages, but now we have only two dirty pages and one clean , shared page. Two other minor things, i want to go over is that how security effects dylb, these two big security things, that have impacted dylb. 

So one is ASLR , address space layout randomization, this is a decade or two old technology,where basically you randomiz the load address. The second is code signing, it has to, many of you have had  to deal with code signing, In Xcode, and you think of code signing as, you run a cryptographic hash over the entire file, and then sign it with your signature. Well, in order to validate that run time, that means the entire file would have to be re-read. So instead what actually happens at build time, is every single page of your Mach-O file gets its own individual cryptographic hash. And all those hashes are stored in the LINKEDIT. This allows each page to be validated that it hasn't been tampered with and was owned by you at page in time. So we finished the crash course ,now I'm going to walk you from exec to main. 

### what is exec?

Exec is a system call. When you trap into the kernel, you basically say I want to replace this process with this new program. The kernel wipes the entire address space and maps in that executable you specified. Now for ASLR it maps it in at a random address. The next thing it does is from that random, back down to zero, it marks that whole region inaccessible, ad by that I mean it's marked not readable, not writeable, not exetutable. the size of that region is at least 4kb to 32bit processes and at least 4GB for 64 bit processes. this catches any NULL prointer references and also foresees more bits, it catches any , pointer truncations. Now life was easy for the first couple decades, of Unix because all I do is map a program, set the PC into it, and start running it. And then shared libraries were invented. So who loads dylibs? the quickly realize that they got really complicated fast and the kernel people didn't want the kernel to do it, so instead a helper program was created. In our platform it's called dylb. On other Unix's you may know it as LB.SO. So when the kernel's done mapping a progcess it now mpas another Mach-O called dyld into that process at another random address. Set the PC into dyld and let's dyld finish launching the process. So now dyld's running in process and its job is to load all the dylibs that you denpend on and get everything prepared and running. So let's walk through those seteps. this is a whole bunch of steps and it has sort of a timeline along the bottom here, as we walk through these we'll walk through the timeline. so first thing , is dyld has to map all the dependent dylibs. well what are the dependents dylibs. to find those it first reads the header of the main executable that the kernel already maped in that header is a list of all the dependent libraries. So it's got to parse that out. then it has to find each dylib. And once it's found each dylib it has to open and run the start of each file it needs to make sure that it is a Mach-O file, validate it , find its code signature, register that code signature to the kernel And then it can actually call mmap at each segment in that dylib . okay so that's pretty simple. Your app knows about the kernel dyld, dyld then says oh this app depends on A and B dylib, load the two of those we're done . Well, it gets more complicated , because A.dylib and B.dylib themselves could depend upon the dylibs.   






