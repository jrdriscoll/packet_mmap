These are files used to help understand the implementation of `AF_PACKET` with shared `TX` rings.

There is a test program `packet_mmap.c` that can be used to write data using `AF_PACKET`.   I believe this to be the original test program for the initial implementation of rx mmap, which (for now) I have only modified to make it build without warning.  You can find the original [here](https://web.archive.org/web/20120317094808/http://wiki.ipxwarzone.com/index.php5?title=Linux_packet_mmap).

The file `status.stp` is a [`SystemTap`](https://sourceware.org/systemtap/) script that provides some information about the functioning of `net/packet/af_packet.c`.  Because it refers to line numbers, it is specific to kernel version 2.13.0-166 (well, probably not that specific as `af_packet.c` isn't changed every day).

Because of optimization, `SystemTap` may not be able to locate arguments or variables.  In particular, because `af_packet.c` provides all of the `AF_PACKET` functionality within that single translation unit, most routines have been inlined by optimization making it difficult to trace what is happening in `af_packet.c`.

Because of this, it is helpful to build a version of the kernel where `af_packet.c` has been compiled with no optimization.  To do this you will need to obtain a copy of the kernel source, adjust the `Makefile` for `af_packet.h`, build the kernel, install the kernel, and finally, boot the kernel.

I happen to be doing this using Ubuntu desktop.  A recipe for downloading kernel source and building on Ubuntu can be found [here](https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel).  Note that you will want to build with debug symbols, and that you will need to intall the debugging information with `sudo dpgk -i linux-<whatever-version>.ddeb` (which is not mentioned in the "recipe").

In order to build `af_packet.c` without optimization, add the line `CFLAGS_af_packet.o = -O0` to the `Makefile` in `net/packet`.

If you have never built and installed a kernel before, you might want to do this for the first time using an unimportant machine or in a VM.  And you might want to change your grub configuration to show the menu of kernels at boot.

Once you have successfully booted the kernel with debug information and `af_packet.c` compiled without optimization, you can run the `SystemTap` script with `sudo stap status.stp` and then run `sudo ./packet_mmap -c 2 eth0` to send 2 frames via a shared TX ring to `eth0` (the supplied `Makefile` will build `packet_mmap`).  As the frames are processed the `SystemTap` script should print out diagnostic information.
