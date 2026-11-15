# Nebulax16 ISA Specification

## Overview

|Property|Value|
|-|-|
|Data width|16 bits|
|Instruction width|16 bits|
|General-purpose registers|8 (R0-R7), 16 bits each|
|Zero register|R0 reads 0; R0 writes are ignored|
|Address space|Word-addressed (one address = one 16-bit word)|
|Memory architecture|Harvard|
|Execution model|Single-cycle|
|Instruction count|16 opcodes|
|Arithmetic|16-bit two's complement, wrapping (no flags)|

## Registers

|Register|Encoding|Description|
|-|-|-|
|R0|000|Constant zero (hardwired)|
|R1-R7|001-111|General purpose|
|PC|n/a|Program counter|
|OUT|n/a|16-bit output register|

## Memory

|Memory|Width|Access|Size|
|-|-|-|-|
|Instruction memory|16 bits|Read only|Preloaded with the program|
|Data memory|16 bits|LOAD / STORE|256 words|

Instruction Formats
Bit:       15-12   11-9   8-6   5-3   2-0
R-type:    opcode  rd     rs1   rs2   unused
I-type:    opcode  rd     rs1   imm6
S/B-type:  opcode  rs2    rs1   imm6
J-type:    opcode  addr12

|Format|Used by|
|-|-|
|R-type|ADD, SUB, AND, OR, XOR, SLL, SRL, SLT|
|I-type|ADDI, LOAD, OUT|
|S/B-type|STORE, BEQ|
|J-type|JMP|
|No operand|NOP, HALT|

imm6 is a signed 6-bit value, sign-extended to 16 bits (-32 to +31). Unused bits are 0.

## Instruction Set

|Opcode|Mnemonic|Format|Syntax|Operation|
|-|-|-|-|-|
|0000|ADD|R|`ADD rd, rs1, rs2`|rd = rs1 + rs2|
|0001|SUB|R|`SUB rd, rs1, rs2`|rd = rs1 - rs2|
|0010|AND|R|`AND rd, rs1, rs2`|rd = rs1 \& rs2|
|0011|OR|R|`OR rd, rs1, rs2`|rd = rs1 \| rs2|
|0100|XOR|R|`XOR rd, rs1, rs2`|rd = rs1 ^ rs2|
|0101|SLL|R|`SLL rd, rs1`|rd = rs1 << 1|
|0110|SRL|R|`SRL rd, rs1`|rd = rs1 >> 1 (logical)|
|0111|SLT|R|`SLT rd, rs1, rs2`|rd = (rs1 < rs2) ? 1 : 0 (unsigned)|
|1000|ADDI|I|`ADDI rd, rs1, imm`|rd = rs1 + sext(imm6)|
|1001|LOAD|I|`LOAD rd, imm(rs1)`|rd = DMEM\[rs1 + sext(imm6)]|
|1010|STORE|S|`STORE rs2, imm(rs1)`|DMEM\[rs1 + sext(imm6)] = rs2|
|1011|BEQ|B|`BEQ rs1, rs2, offset`|if (rs1 == rs2) PC = PC + 1 + sext(imm6), else PC = PC + 1|
|1100|JMP|J|`JMP addr`|PC = addr12|
|1101|OUT|I|`OUT rs1`|OUT = rs1|
|1110|NOP|none|`NOP`|no operation|
|1111|HALT|none|`HALT`|stop; PC stops updating|

All instructions except BEQ (taken), JMP, and HALT set PC = PC + 1.

## Notes

* SLL and SRL shift by exactly 1 bit; the rs2 field is ignored.
* SLT compares as unsigned values.
* BEQ offset is in instructions, relative to the instruction after the branch (-32 to +31).
* JMP address range is 0 to 4095.
* STORE and BEQ put rs2 in bits 11-9 and rs1 in bits 8-6.
* OUT encoding: `1101 000 rs1 000000`.

## Pseudo-Instructions

|Pseudo|Expands to|
|-|-|
|MOV rd, rs|ADD rd, rs, R0|
|CLR rd|ADD rd, R0, R0|
|LI rd, imm|ADDI rd, R0, imm (imm in -32..31)|
|INC rd|ADDI rd, rd, 1|
|DEC rd|ADDI rd, rd, -1|

## Encoding Examples

|Assembly|Binary|Hex|
|-|-|-|
|ADD R3, R1, R2|0000 011 001 010 000|0x0650|
|SUB R4, R2, R1|0001 100 010 001 000|0x1888|
|ADDI R1, R0, 5|1000 001 000 000101|0x8205|
|ADDI R1, R1, -1|1000 001 001 111111|0x827F|
|LOAD R4, 2(R0)|1001 100 000 000010|0x9802|
|STORE R3, 2(R0)|1010 011 000 000010|0xA602|
|BEQ R1, R2, +2|1011 010 001 000010|0xB442|
|JMP 5|1100 000000000101|0xC005|
|OUT R4|1101 000 100 000000|0xD100|
|NOP|1110 000000000000|0xE000|
|HALT|1111 000000000000|0xF000|

Test Programs
0  0x8205  ADDI R1, R0, 5
1  0x8403  ADDI R2, R0, 3
2  0x0650  ADD  R3, R1, R2
3  0xF000  HALT

Expected: R1 = 5, R2 = 3, R3 = 8.

Store, load, output
0  0x8205  ADDI R1, R0, 5
1  0x8403  ADDI R2, R0, 3
2  0x0650  ADD  R3, R1, R2
3  0xA602  STORE R3, 2(R0)
4  0x9802  LOAD R4, 2(R0)
5  0xD100  OUT R4
6  0xF000  HALT

Expected: R3 = 8, R4 = 8, DMEM\[2] = 8, OUT = 8.

Countdown loop
0  0x8203  ADDI R1, R0, 3
1  0x827F  ADDI R1, R1, -1
2  0xB041  BEQ  R1, R0, +1
3  0xC001  JMP  1
4  0xF000  HALT

Expected: R1 = 0.

## Reset

PC = 0, all registers = 0, OUT = 0, halt cleared. Data memory is not cleared.

## Limitations

* 6-bit immediates (-32 to +31)
* Shifts by 1 bit only
* BEQ is the only conditional branch
* No multiply, divide, or call/return
* No carry, overflow, or negative flags
* No interrupts, exceptions, or pipelining

## Version His

|Version|Date|Changes|
|-|-|-|
|1.0|2026-11-15|Initial spec|



