# assembler

The assembler is baked into skiff for handling asm code. 

```

+-----------------+
|                 |
|                 |
|  Instruction    |
|                 |
|  Generator      |          +-------------+-------------+
|                 |          |                           |
|                 |          |  Binary Generators        |
+--^-----------+--+          |                           |
   |           |             +-----^-------------+-------+
   |          Ins                  |             |
   |        Bytecode               |             |
   |           |                   |             |
+--+-----------v--+            Bytecode          |
|                 |                |             |
|                 +----------------+         Generated
|  Assembler      |                          Binary
|                 |                              |
|                 |                              |
|                 <------------------------------+
+--^----------+---+
   |          |
   |          |
 File       Binary
   |          |
+--+----------v--+
|                |
|    Caller      +------------> output.bin
|   (skiffd)     |
+-----^----------+
      |
      |
   input.asm



```


# ASM Directives

##  .init

**Description:**
*Required* Used to define the name of the first label to start execution from. 

**Example:**
```
  .init main
```

## .debug  

| Level | Meaning |
|----|----|
| 0 | No debug, same as not being defined |
| 1 | Minimal debug |
| 2 | Moderate debug |
| 3 | Extreme debug |

**Description:**
Indicates to the VM that debug mode should be enabled at a specified level. This directive is not required, but when set will force output debug messages
at runtime.

**Example:**
```
  .debug 1
```

## .string

**Description:**
Defines a string 

**Example:**
```
  .string string_name "Body of the string"
```

Encoded strings in the data section take the form: 

```
	<unsigned 16-bit 'length'> <string data> 
```

## .i8 .. .i64, .u8 .. u64 

**Description:**
Defines a specifically sized integer. The memory of constants at runtime are word aligned 
so the minimum size of a constant is 2 bytes. This means that constants such as i8 and u8
are 2 bytes, and strings may have a trailing byte.

**Example:**
```
  .i8 int_one 42
  .i16 int_two 33
  .i32 int_three 99
  .i64 int_four 594

  .u8 unsigned_one 0
  .u16 unsigned_two 1
  .u32 unsigned_three 255
  .u64 unsigned_four 99000
```

## .float 

**Description:**
Defines a floating point number

**Example:**
```
  .float pi 3.14159
```

### Note about data directives
Upon init of the binary the data directives are loaded into memory slot 0 in a word aligned manner. 
See the section regarding memory access and data storage / retrieval.



## .code

**Description:**
Indicates the start of 'code' space. No more directives shall follow this

**Example:**
```
  .code
```

# Symbol Usage

| Symbol | Description |
|----|----
| & | Address of constant / label
| $ | Value of constant
| # | Length of constant 
| @ | Raw 64-bit value

# Instructions

## Labels

**Format:** [label_name]:
**Description:** Labels mark locations within instructions, and have no direct encoding within the instruction space. 
**Example:** `my_label:`

## Macros

Not really an instruction, but a way to ease the burden of writing asm programs, a macro takes the following forms:

```
#macro PRINT_CODE_ASCII "mov i3 @9"
#macro M_EXIT "mov i0 @0" \
              "exit"
```

Once the macros are created they can be used anywhere within the code as in the following example:

```
.init main
#macro M_EXIT "mov i0 @0" \
              "exit"
.code
main:
	#M_EXIT
```

which is equivalent to : 

```
.init main
.code
main:
	mov i0 @0
	exit
```

# Example program

```
.init main  ; Indicate that the entry point
.code       ; Declare that instructions are next

; MODULUS
; i0 = i1 % i2
; Uses i0 - i4
modulus:
	div i3 i1 i2
  mov i4 @0
	beq i3 i4 moduiz
	jmp moduinz
moduiz:
    mov i0 @0
		jmp moddone
moduinz:
		mul i3 i2 i3
		sub i0 i1 i3
		jmp moddone
moddone:
    ret

main:
  mov i1 @13
  mov i2 @5
  call modulus
  exit
```