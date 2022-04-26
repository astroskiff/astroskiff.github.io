# What is Astroskiff? 

What here is referred to as Astroskiff, is in fact, ASTRO/Skiff, or as I've recently taken to calling it, ASTRO plus Skiff. Astro is a high level language (HLL) that is currently being planned that will run on the Skiff Virtual Machine.
Currently, the Skiff VM is under heavy development as its only about month old (as of April 26, 2022). Once Skiff is solidified a bit and seems performant enough Astro will be started. 

# Skiff Virtual Machine (SVM)

*Quick Links*

[SVM Instruction Set Doc](https://github.com/astroskiff/astroskiff.github.io/blob/main/docs/instructions.md)

[SVM Binary Encoding Doc](https://github.com/astroskiff/astroskiff.github.io/blob/main/docs/binary_encoding.md)

[SVM Assembler Doc](https://github.com/astroskiff/astroskiff.github.io/blob/main/docs/assembler.md)

[Skiff Repo](https://github.com/astroskiff/skiff)



The computational model that skiff is based off of is that of a register based reduced instruction set machine. Within the [skiff project](https://github.com/astroskiff/skiff) there exists an assembler that can generate programs written in the skiff assembly as detailed by the [instruction set document](https://astroskiff.github.io/docs/instructions.md), and a list of example programs can be found [here](https://github.com/astroskiff/skiff/tree/main/vm_programs). 

## VM Layout

The SVM has a set of integer registers, floating point registers, and miscellaneous registers that are used for computations along with a stack and "slotted" memory system. The integer and floating point registers maintain
the same underlying data storage mechanisms but are separated out by name to help enforce convention. 

### Definitions / Conventions 

The SVM has a notion of `word`, `double word`, and `quadruple word` that will be seen in the instruction set. These refer to a `2-byte`, `4-byte`, and `8-byte` value respectively. 

### Registers

Here is a list of the SVMs registers, their representation in SVM bytecode, along with a description of what the register is used for.

|    Register  |  Byte code  |  Description  |
|----|----|----|
| x0 | 0x00 | '0' constant |
| x1 | 0x01 | '1' constant |
| ip | 0x02 | Instruction pointer |
| sp | 0x03 | Stack pointer |
| i0 | 0x10 | Integer Register |
| i1 | 0x11 | Integer Register |
| i2 | 0x12 | Integer Register |
| i3 | 0x13 | Integer Register |
| i4 | 0x14 | Integer Register |
| i5 | 0x15 | Integer Register |
| i6 | 0x16 | Integer Register |
| i7 | 0x17 | Integer Register |
| i8 | 0x18 | Integer Register |
| i9 | 0x19 | Integer Register |
| f0 | 0x20 | Floating Point Register |
| f1 | 0x21 | Floating Point Register |
| f2 | 0x22 | Floating Point Register |
| f3 | 0x23 | Floating Point Register |
| f4 | 0x24 | Floating Point Register |
| f5 | 0x25 | Floating Point Register |
| f6 | 0x26 | Floating Point Register |
| f7 | 0x27 | Floating Point Register |
| f8 | 0x28 | Floating Point Register |
| f9 | 0x29 | Floating Point Register | 
| op | 0xFF | Operation result        | 

### Data Directives

While this is more assembler-specific information, I think it warrants showing here to get an idea of the base types that the SVM is equipped to handle. 

|  Directive |  Meaning  |
|----|----|
|  .u8  | Unsigned 8-bit integer |
|  .u16 | Unsigned 16-bit integer|
|  .u32 | Unsigned 32-bit integer|
|  .u64 | Unsigned 64-bit integer|
|  .i8  | Signed 8-bit integer |
|  .i16 | Signed 16-bit integer|
|  .i32 | Signed 32-bit integer|
|  .i64 | Signed 64-bit integer|
| .float| Floating point number|
| .string| ASCII String data |

### Stack

The system stack is a straight forward stack that can push/pop words, double words, and quad words. When the stack grows so too does the value in the `sp` register. Using this register you can see how much data remains in the stack. While some machines allow indexing into the stack at specific offsets, at the time of this writing (April 26, 2022) there is no convention for doing so. This was a decision made to promote the use of slotted memory (below) and to keep the isntruction set small. The stack as it exists today is envisioned to be used for temporarily storing information during function calls or computations. 

### Memory 

Standard systems have a heap and a stack that are arbitrarily defined in physical memory. Emulating this would have been one way to work with memory in the VM, but it was decided that making operations simpler would be preferred. Instead, the memory model in skiff operates as follows:

0) A number of bytes is requested for allocation via the `alloc` instruction
1) The requested number of bytes is allocated and an id is returned into a specified slot
2) Using this id `store` and `load` instructions can perform their respective operations on data
3) When the memory is no longer needed the `free` instruction can be utilized to mark the memory for reuse.

An example:

```

+---+   +---+---+---+---+
|   |   |   |   |   |   |
| 0 +-->|   |   |   |   |
|   |   |   |   |   |   |
+---+   +---+---+---+---+

When the program starts, any data declared via directives is slotted into memory id 0

If the `alloc` command is ran twice, with the first requestin 7 bytes, and the second requesting 2, it will return ids 1 and 2 repectively.

+---+   +---+---+---+---+
|   |   |   |   |   |   |
| 0 +-->|   |   |   |   |
|   |   |   |   |   |   |
+---+   +---+---+---+---+---+---+---+
|   |   |   |   |   |   |   |   |   |
| 1 +-->|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |   |   |
+---+   +---+---+---+---+---+---+---+
|   |   |   |   |
| 2 +-->|   |   |
|   |   |   |   |
+---+   +---+---+

Using the given ids `store` and `load` could move memory from/to registers from any of the above slots (0-2). Once memory is no longer required, `free` could be ran. For the sake of example lets `free` slot 1.

free_slot_one:
  mov i0 @1   ; Load 1 into integer register 0
  free i0     ; Request memory from slot 1 be freed

+---+   +---+---+---+---+
|   |   |   |   |   |   |
| 0 +-->|   |   |   |   |
|   |   |   |   |   |   |
+---+   +---+---+---+---+
|   |
| 1 |
|   |
+---+   +---+---+
|   |   |   |   |
| 2 +-->|   |   |
|   |   |   |   |
+---+   +---+---+

Slot 1 is no longer pointing to anything and the id is marked by the system for reuse. Now the next time `alloc` is called the id 1 will be utilized for storage. Lets say for example we request memory to allocate 10 bytes for us as so:

allocate_ten_bytes:
  mov i1 @10    ; Load 10 into integer register 1
  alloc i0 i1   ; Request an allocation of 10 bytes and store the resulting id into integer register 0
  ret

Memory will now be as follows:

+---+   +---+---+---+---+
|   |   |   |   |   |   |
| 0 +-->|   |   |   |   |
|   |   |   |   |   |   |
+---+   +---+---+---+---+---+---+---+---+---+---+
|   |   |   |   |   |   |   |   |   |   |   |   |
| 1 +-->|   |   |   |   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |   |   |   |   |   |
+---+   +---+---+---+---+---+---+---+---+---+---+
|   |   |   |   |
| 2 +-->|   |   |
|   |   |   |   |
+---+   +---+---+

With the number `1` residing in `i0`

```

The slotted memory model not only makes keeping track of variable memory super simple, but it enables communication to devices really easy.


### Interrupts

The system can have any number of interrupts, determined by the programmer. 
Labels with a name matching `interrupt_N` where `N` is any number `[0, uint64_t::max()]` 
will be read in and marked in the system as a memory location that can be called from an interrupt. 

Devices and system calls that perform interrupts will take in the interrupt number
it is required to call.


**Example**

```
.init fn_main
.code
interrupt_0:
  dirq  ; Disable interrupts
  nop
  eirq  ; Enable interrupts
  ret

interrupt_1:
  ret

interrupt_99:
  ret

fn_main:
  exit
```

The above example shows how, by the use of label naming, you can declare what interrupts you want to exist. Them being present will cause the
assembler to map the number in the label to the address in memory that the label is. This allows you to pass the number declared in the label name
to some external device that will interrupt when processing is complete. 

When an interrupt is fired it is similar to a call instruction being executed, this means that at the end of the section handling the interrupt, 
a `ret` should exist to return to wherever the code was interrupted from. 

Using `dirq` and `eirq` you can disable or enable interrupt requests. It is recommended at the beginning of an interrupt handle to disable interrupts
so you can be sure to process whatever is going on, though its not required. 

If an interrupt is fired externally while interrupts are disabled then the request will be ignored. It is up to the interrupter to decide if it wants to
try again later or not as interrupt requests are not queued... they are either accepted or denied.

## System Calls

The actual system calls that exist are under development, but there is a means to extend the VM via system `devices` that are called on via a system call. 
More about system calls will be documented here as they are solidifed.