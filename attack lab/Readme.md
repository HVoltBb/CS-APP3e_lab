# Solving strategy for phase 5 (return oriented programming)

Phase 1 through 4 are fairly easy to do. The only place to keep an eye on is the byte order and any typos. As least for me, typing in small endian order is not natural, and I tend to accidentally reverse some hex numbers while reading them off the screen and typing them out.

Phase 5 is kind of hard, because not all the gadgets you need are properly explained in the handout (at least not in [this one](http://csapp.cs.cmu.edu/3e/attacklab.pdf) 5-25-2019). 


First, let's review the return oriented programming (ROP). It is a new concept to me, but if you are familiar with it, you can skip this paragraph. ROP is a method to circumvent the stack randomization (or more formally address space layout randomization [ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization)) in order to take advantage of the buffer overflow bug. I am a person fond of analogies, and I often find they are useful in helping me understand new things. Essentially, what ROP is trying to accomplish is similar to assembling a meaningful sentence from a pool of cutout letters from newspapers or magezines, what a criminal would send as a ransom letter for example. Each of these cutout letters represents a gadget that we use to assemble a meaningful sequence of instructions in our case. We are trying to do the same thing here in phase 4 and 5.

![Cutout letters](https://github.com/HVoltBb/HVoltBb.github.io/blob/master/pics/ace.jpg)

Here is the solving strategy. Things we need to do:
1. Write the string (cookie) somewhere


	a. Where to put the string?


	b. How to encode the string? 


2. Move the location of the string to %rdi


	c. How to get the string location with ASLR, hardcode the address or compute it?


	d. Move it directly or through other registers?


So, only two things to accomplish, and it is not that hard. Let's examine in details the items we need to do. 


Item (a): The stack is writable, but it is randomized. So, we cannot hardcode the address. Since we are taking advantage of the buffer overflow bug, we can freely write the space between the current starting address of the buffer (saved in %esp in this case), moving up in address space, to the canary bytes. We can't write to the lower address space or beyond the canary bytes of function _test_. It is a __miracle__ that function _getbuf_ doesn't have the canary bytes, or is it so contrived that we can take advantage of the buffer overflow?


So we need to write the cookie on the stack below the return address of _getbug_, and the exact location is unknown so far, which depends on the gadgets we have.


Item (b): Phase 4 instructions are pretty clear on this, and we do not explore in here.


Item (c): No hardcoding because of ASLR, obviously. But the relative location on the stack is fixed. We just need to get the cookie to some place with a fixed distance to a known location on the stack. The easiest known location is where %rsp is pointing to in this case (I don't see %rbp in _getbuf_, but the frame point, if present, is also a good candidate).

So, we need to compute an address, some known bytes away from where %rsp is pointing to, more precisely, a higher address from that in %rsp (due to Item a). So, we need an _add_ gadget, or _lea_ gadget, and we need to add more than a few quadwords to %rsp. There are two add-one functions in the farm, but they do not fit our bill.


Next, we need to consult the [Intel X64 instructions manual](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf). We check ADD first, and if we can't find any gadget there then check LEA. Go to page 133 for the ADD instruction. Go through the Opcode column, and search through the gadget farm for candidates. You may need to use gcc and objdump to get the opcode to look for, because the documentation is very hard to read but a realy good place to start looking. You don't need to go down that list too far to find a good gadget. LEA is on page 627, but you really don't need to go there.


Now, we have enough knowledge to do phase5 fairly quickly. Don't stare at Figure 3 of the handout, if you can't figure out phase5, because it is not enough. To solve phase5, you need a systematic approach as shown above. This way you are covering enough ground and going in the right direction.


Good luck!


Can Z 