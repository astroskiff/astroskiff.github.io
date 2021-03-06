# Instructions + Assembler Information

Some of the information here is duplicated across the other documentation but I wanted to make sure that the information was available in one place for quick reference. 
In the event that the documents diverge use this as the go-to for information regarding system layout and instructions, and then please make a PR to fix the discrepancy.

## Registers:

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

## nop
**Opcode:** 0x00
**Instruction Layout:**
```
	[ Opcode ]
	0000 0000 
```
**Format:** `nop`
**Description:** A no-operation. This instruction does nothing.
**Example:**	`nop`

## exit
**Opcode** 0x01
**Instruction Layout:**
```
	[ Opcode ]
	0000 0000 
```
**Format:** `exit`
**Description:** Exit execution 
**Example:**	`exit`

## blt
**Opcode** 0x02
**Instruction Layout:**
```
	[ Opcode ] [ LHS Reg ] [ RHS Reg ] 
	0000 0000 | 0000 0000 | 0000 0000 
	
	[ ---------------- Address ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `blt <register lhs> <register rhs> <label>`
**Description:** Compare two integer registers. Branch to given address iff `lhs < rhs`
**Example:**	`blt i0 i1 label_name	; Branch to a label`
**Note:** In the future this instruction should be updated to allow the
currently empty byte to be a `variant` byte, and allow registers to be
used in the stead of labels. That way we can branch further than u32

## bgt
**Opcode** 0x03
**Instruction Layout:**
```
	[ Opcode ] [ LHS Reg ] [ RHS Reg ] 
	0000 0000 | 0000 0000 | 0000 0000 
	
	[ ---------------- Address ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `bgt <register lhs> <register rhs> <label>`
**Description:** Compare two integer registers. Branch to given address iff `lhs > rhs`
**Example:**	`blt i0 i1 label_name	; Branch to a label`
**Note:** In the future this instruction should be updated to allow the
currently empty byte to be a `variant` byte, and allow registers to be
used in the stead of labels. That way we can branch further than u32

## beq
**Opcode** 0x04
**Instruction Layout:**
```
	[ Opcode ] [ LHS Reg ] [ RHS Reg ] 
	0000 0000 | 0000 0000 | 0000 0000 
	
	[ ---------------- Address ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `beq <register lhs> <register rhs> <label>`
**Description:** Compare two integer registers. Branch to given address iff `lhs = rhs`
**Example:**	`beq i0 i1 label_name	; Branch to a label`
**Note:** In the future this instruction should be updated to allow the
currently empty byte to be a `variant` byte, and allow registers to be
used in the stead of labels. That way we can branch further than u32

## jmp
**Opcode** 0x05
**Instruction Layout:**
```
	[ Opcode ]
	0000 0000  
	
	[ ---------------- Address ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `jmp <label>`
**Description:** Unconditionally jump to a label.
**Example:**	`jmp label_name`
**Note:** In the future this instruction should be updated to allow one
of the currently empty byte to be a `variant` byte, and allow registers 
to be used in the stead of labels. That way we can branch further than u32

## call
**Opcode** 0x06
**Instruction Layout:**
```
	[ Opcode ]
	0000 0000  
	
	[ ---------------- Address ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `call <label>`
**Description:** Unconditionally go to a label, pushing the next address to the call stack. This will allow a return to the next instruction from the call destination.
**Example:**	`call label_name`

## ret
**Opcode** 0x07
**Instruction Layout:**
```
	[ Opcode ]
	0000 0000 
```

**Format:** `ret`
**Description:** Return to the next address in the call stack. If the call stack is empty, execution will be halted.
**Example:**	`ret`

## mov
**Opcode** 0x08
**Instruction Layout:**
```
	[ Opcode ]  [ Dest ]
	0000 0000 | 0000 0000 
	
	[ ------------------ Value ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `mov <register> <source>`
**Description:** Move some data into a given register.

**Example:**	
```
	mov i0 &my_int		; Address of constant
	mov i0 &label			; Label address
	mov i0 #my_int		; Length of constant (in words)
	mov i0 @33				; Move raw value into i0
```

## add
**Opcode** 0x09
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `add <dest register> <lhs register> <rhs register>`
**Example:**	`add i0 i0 i1`
**Description:** Adds the value of two integer registers.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## sub
**Opcode** 0x0A
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `sub <dest register> <lhs register> <rhs register>`
**Example:**	`sub i0 i0 i1`
**Description:** Subtracts the value of two integer registers.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## div
**Opcode** 0x0B
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `div <dest register> <lhs register> <rhs register>`
**Example:**	`div i0 i0 i1`
**Description:** Divides the value of two integer registers.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## mul
**Opcode** 0x0C
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `mul <dest register> <lhs register> <rhs register>`
**Example:**	`mul i0 i0 i1`
**Description:** Multiplies the value of two integer registers.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## addf
**Opcode** 0x0D
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `addf <dest register> <lhs register> <rhs register>`
**Example:**	`addf f0 f0 f1`
**Description:** Adds the value of two floating point registers.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## subf
**Opcode** 0x0E
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `subf <dest register> <lhs register> <rhs register>`
**Example:**	`subf f0 f0 f1`
**Description:** Subtracts the value of two floating point registers.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## divf
**Opcode** 0x0F
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `divf <dest register> <lhs register> <rhs register>`
**Example:**	`divf f0 f0 f1`
**Description:** Divides the value of two floating point registers.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## mulf
**Opcode** 0x10
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `mulf <dest register> <lhs register> <rhs register>`
**Example:**	`mulf f0 f0 f1`
**Description:** Multiplies the value of two floating point registers.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## lsh
**Opcode** 0x11
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `lsh <dest register> <lhs register> <rhs register>`
**Example:**	`lsh i0 i0 i1`
**Description:** Left shifts the value of a register by the value of another.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## rsh
**Opcode** 0x12
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `rsh <dest register> <lhs register> <rhs register>`
**Example:**	`rsh i0 i0 i1`
**Description:** Right shifts the value of a register by the value of another.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## and
**Opcode** 0x13
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `and <dest register> <lhs register> <rhs register>`
**Example:**	`and i0 i0 i1`
**Description:** Logical AND the value of a register by the value of another.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## or
**Opcode** 0x14
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `or <dest register> <lhs register> <rhs register>`
**Example:**	`or i0 i0 i1`
**Description:** Logical OR the value of a register by the value of another.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## xor
**Opcode** 0x15
**Instruction Layout:**
```
	[ Opcode ] [ Dest Reg ] [ LHS Reg ] [ RHS Reg ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `xor <dest register> <lhs register> <rhs register>`
**Example:**	`xor i0 i0 i1`
**Description:** Logical XOR the value of a register by the value of another.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## not
**Opcode** 0x16
**Instruction Layout:**
```
	 [ OpCode ] [ Dest Reg ] [  Reg   ]
	0000 0000  | 0000 0000  | 0000 0000
```
**Format:** `not <dest register> <register>`
**Example:**	`not i0 i0`
**Description:** Logical NOT the value of a register.
**Note:** Future expansion of this instruction is expected such that raw values could be encoded and a variant byte could be introduced to indicate that its not a register-only instruction. This would have potential execution speed ramifications. 

## bltf
**Opcode** 0x17
**Instruction Layout:**
```
	[ Opcode ] [ LHS Reg ] [ RHS Reg ] 
	0000 0000 | 0000 0000 | 0000 0000 
	
	[ ---------------- Address ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `bltf <register lhs> <register rhs> <label>`
**Description:** Compare two float registers. Branch to given address iff `lhs < rhs`
**Example:**	`bltf i0 i1 label_name	; Branch to a label`
**Note:** In the future this instruction should be updated to allow the
currently empty byte to be a `variant` byte, and allow registers to be
used in the stead of labels. That way we can branch further than u32

## bgtf
**Opcode** 0x18
**Instruction Layout:**
```
	[ Opcode ] [ LHS Reg ] [ RHS Reg ] 
	0000 0000 | 0000 0000 | 0000 0000 
	
	[ ---------------- Address ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `bgt <register lhs> <register rhs> <label>`
**Description:** Compare two float registers. Branch to given address iff `lhs > rhs`
**Example:**	`blt i0 i1 label_name	; Branch to a label`
**Note:** In the future this instruction should be updated to allow the
currently empty byte to be a `variant` byte, and allow registers to be
used in the stead of labels. That way we can branch further than u32

## beqf
**Opcode** 0x19
**Instruction Layout:**
```
	[ Opcode ] [ LHS Reg ] [ RHS Reg ] 
	0000 0000 | 0000 0000 | 0000 0000 
	
	[ ---------------- Address ---------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```

**Format:** `beq <register lhs> <register rhs> <label>`
**Description:** Compare two float registers. Branch to given address iff `lhs = rhs`
**Example:**	`beq i0 i1 label_name	; Branch to a label`
**Note:** In the future this instruction should be updated to allow the
currently empty byte to be a `variant` byte, and allow registers to be
used in the stead of labels. That way we can branch further than u32

## aseq 
**Opcode** 0x1A
**Instruction Layout:**
```
	[ Opcode ] [ Expected ] [  Actual  ]
 	 0000 0000 | 0000 0000  | 0000 0000 
```
**Format:** `aseq <expected register> <actual register>`
**Example:**	`aseq i0 i2`
**Description:** Assert that the value in the `expected register` matches that of the value in the `actual register`. 
Failure of this condition will lead the VM to exit with a code of `1`

## asneq 
**Opcode** 0x1B
**Instruction Layout:**
```
	[ Opcode ] [ Expected ] [  Actual  ]
 	 0000 0000 | 0000 0000  | 0000 0000 
```
**Format:** `asneq <expected register> <actual register>`
**Example:**	`asneq i0 i2`
**Description:** Assert that the value in the `expected register` does not match that of the value in the `actual register`. 
Failure of this condition will lead the VM to exit with a code of `1`

## push_w
**Opcode** 0x1C
**Instruction Layout:**
```
	[ Opcode ] [ Source ]
  0000 0000 | 0000 0000 
```
**Format:** `push_w <source register>`
**Example:**	`push_w i0`
**Description:** Push a word to the stack.
After this operation, the stack pointer will be updated
to reflect the size of the stack. 
Failure to push data to stack will result in runtime error.

## push_dw
**Opcode** 0x1D
**Instruction Layout:**
```
	[ Opcode ] [ Source ]
  0000 0000 | 0000 0000 
```
**Format:** `push_dw <source register>`
**Example:**	`push_dw i0`
**Description:** Push a double-word to the stack.
After this operation, the stack pointer will be updated
to reflect the size of the stack
Failure to push data to stack will result in runtime error.

## push_qw
**Opcode** 0x1E
**Instruction Layout:**
```
	[ Opcode ] [ Source ]
  0000 0000 | 0000 0000 
```
**Format:** `push_qw <source register>`
**Example:**	`push_qw i0`
**Description:** Push a quad-word to the stack.
After this operation, the stack pointer will be updated
to reflect the size of the stack
Failure to push data to stack will result in runtime error.

## pop_w
**Opcode** 0x1F
**Instruction Layout:**
```
	[ Opcode ] [ Dest ]
  0000 0000 | 0000 0000 
```
**Format:** `pop_w <dest register>`
**Example:**	`pop_w i0`
**Description:** Pop a word from the stack.
After this operation, the stack pointer will be updated
to reflect the size of the stack

## pop_dw
**Opcode** 0x20
**Instruction Layout:**
```
	[ Opcode ] [ Dest ]
  0000 0000 | 0000 0000 
```
**Format:** `pop_dw <dest register>`
**Example:**	`pop_dw i0`
**Description:** Pop a double-word from the stack.
After this operation, the stack pointer will be updated
to reflect the size of the stack

## pop_qw
**Opcode** 0x21
**Instruction Layout:**
```
	[ Opcode ] [ Dest ]
  0000 0000 | 0000 0000 
```
**Format:** `pop_qw <dest register>`
**Example:**	`pop_qw i0`
**Description:** Pop a quad-word from the stack.
After this operation, the stack pointer will be updated
to reflect the size of the stack

## alloc
**Opcode** 0x22
**Instruction Layout:**
```
	[ Opcode ] | [ Dest Reg ] [ Source] 
	0000 0000  | 0000 0000   | 0000 0000 
```
**Format:** `alloc <dest register> <source register>`
**Example:**	`alloc i0 i8`
**Description:** Allocate n-bytes from source register and retrieve a memory index for new space in dest register. If the
allocation fails, the `op` register will be set to `0`, and `1` otherwise. 

## free
**Opcode** 0x23
**Instruction Layout:**
```
  [ Opcode ]	[  Index ] 
	0000 0000 | 0000 0000
```
**Format:** `free <index register>`
**Example:**	`free i0`
**Description:** Frees the memory index allocated by alloc in its entirety.  If the
free fails, the `op` register will be set to `0`, and `1` otherwise. 

## sw
**Opcode** 0x24
**Instruction Layout:**
```
	[ Opcode ] [ Idx Reg ] [ Offst Reg ] [ Data Reg ]
	0000 0000  | 0000 0000  | 0000 0000 |  0000 0000
```
**Format:** `sw <index register> <offset register> <data register>`
**Example:**	`sw i0 i2 i3`
**Description:** Stores a word in a memory index, at an offset with data from data reg.

## sdw
**Opcode** 0x25
**Instruction Layout:**
```
	[ Opcode ] [ Idx Reg ] [ Offst Reg ] [ Data Reg ]
	0000 0000  | 0000 0000  | 0000 0000 |  0000 0000
```
**Format:** `sdw <index register> <offset register> <data register>`
**Example:**	`sdw i0 i2 i3`
**Description:** Stores a double word in a memory index, at an offset with data from data reg.

## sqw
**Opcode** 0x26
**Instruction Layout:**
```
	[ Opcode ] [ Idx Reg ] [ Offst Reg ] [ Data Reg ]
	0000 0000  | 0000 0000  | 0000 0000 |  0000 0000
```
**Format:** `sqw <index register> <offset register> <data register>`
**Example:**	`sqw i0 i2 i3`
**Description:** Stores a quad word in a memory index, at an offset with data from data reg.

## lw
**Opcode** 0x27
**Instruction Layout:**
```
	[ Opcode ] [ Idx Reg ] [ Offst Reg ] [ Data Reg ]
	0000 0000  | 0000 0000  | 0000 0000 |  0000 0000
```
**Format:** `lw <index register> <offset register> <dest register>`
**Example:**	`lw i0 i2 i3`
**Description:** Loads a word from a memory index, at an offset into the dest register.

## ldw
**Opcode** 0x28
**Instruction Layout:**
```
	[ Opcode ] [ Idx Reg ] [ Offst Reg ] [ Data Reg ]
	0000 0000  | 0000 0000  | 0000 0000 |  0000 0000
```
**Format:** `ldw <index register> <offset register> <dest register>`
**Example:**	`ldw i0 i2 i3`
**Description:** Loads a double word from a memory index, at an offset into the dest register.

## lqw
**Opcode** 0x29
**Instruction Layout:**
```
	[ Opcode ] [ Idx Reg ] [ Offst Reg ] [ Data Reg ]
	0000 0000  | 0000 0000  | 0000 0000 |  0000 0000
```
**Format:** `lqw <index register> <offset register> <dest register>`
**Example:**	`lqw i0 i2 i3`
**Description:** Loads a quad word from a memory index, at an offset into the dest register.

## syscall
**Opcode** 0x2A
**Instruction Layout:**
```
	[ Opcode ]
	0000 0000 

	[ ------------ Callee Address ------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `syscall <unsigned 32-bit number>`
**Description:** Execute a specific system call with the given ID.
**Example:**	`syscall 0` 


## debug
**Opcode** 0x2B
**Instruction Layout:**
```
	[ Opcode ]
	0000 0000 

	[ ------------ Callee Address ------------- ]
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
	0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
```
**Format:** `debug <unsigned 64-bit number>`
**Description:** Prints a debug statement when executed stating the debug ID. 
If `.debug` directive is set more information will follow: 

```
Level 0:
	No extra information

Level 1:
	Instruction pointer

Level 2:
	Level 1 + System registers

Level 3:
	Level 2 + Integer & Floating-point registers
```

**Example:**	`debug 0` 

## eirq
**Opcode** 0x2C
**Instruction Layout:**
```
	[ Opcode ]
 	 0000 0000 
```
**Format:** `eirq`
**Description:** Enables interrupt requests
**Example:**	`eirq` 

## dirq
**Opcode** 0x2D
**Instruction Layout:**
```
	[ Opcode ]
 	 0000 0000 
```
**Format:** `dirq`
**Description:** Disables interrupt requests
**Example:**	`dirq` 

## push_hw
**Opcode** 0x2E
**Instruction Layout:**
```
	[ Opcode ] [ Source ]
  0000 0000 | 0000 0000 
```
**Format:** `push_hw <source register>`
**Example:**	`push_hw i0`
**Description:** Push a half word to the stack.
After this operation, the stack pointer will be updated
to reflect the size of the stack. 
Failure to push data to stack will result in runtime error.

## pop_hw
**Opcode** 0x2F
**Instruction Layout:**
```
	[ Opcode ] [ Dest ]
  0000 0000 | 0000 0000 
```
**Format:** `pop_hw <dest register>`
**Example:**	`pop_hw i0`
**Description:** Pop a half word from the stack.
After this operation, the stack pointer will be updated
to reflect the size of the stack

## shw
**Opcode** 0x30
**Instruction Layout:**
```
	[ Opcode ] [ Idx Reg ] [ Offst Reg ] [ Data Reg ]
	0000 0000  | 0000 0000  | 0000 0000 |  0000 0000
```
**Format:** `shw <index register> <offset register> <data register>`
**Example:**	`shw i0 i2 i3`
**Description:** Stores a half word in a memory index, at an offset with data from data reg.

## lhw
**Opcode** 0x31
**Instruction Layout:**
```
	[ Opcode ] [ Idx Reg ] [ Offst Reg ] [ Data Reg ]
	0000 0000  | 0000 0000  | 0000 0000 |  0000 0000
```
**Format:** `lhw <index register> <offset register> <dest register>`
**Example:**	`lhw i0 i2 i3`
**Description:** Loads a half word from a memory index, at an offset into the dest register.