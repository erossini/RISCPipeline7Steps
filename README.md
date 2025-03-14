# RISC Pipeline simulation



Consider the following assembly code:

```
I1: LOAD r3, M1
I2: MUL r1, r1, r3
I3: LOAD r2, [r2]
I4: LOAD r3, M2
I5: MUL r2, r2, r3
I6: ADD r1, r1, r2
I7: LOAD r2, M3
I8: LOAD r3, M4
I9: ADD r2, r2, r3
I10: MUL r1, r1, r2
```

Here is a diagram showing the out-of-order execution of the code. E.g., processing I7 (EX in cycle 9) can start before processing I5 (RR in cycle 10), 
but we have to be careful not to mess up the program semantics: I7 can go to WB (in cycle 14) only after I6 has read r2 (in cycle 13). 

## Result with one function unit

| Cycle | IF          | ID     | IW     | RR | EX | RB | WB                   | Comments         |
|-------|-------------|--------|--------|----|----|----|----------------------|------------------|
| 1     | I1          |        |        |    |    |    |                      |                  |
| 2     | I2          | I1     |        |    |    |    |                      |                  |
| 3     | I3          | I2     |        |    | I1 |    |                      |                  |
| 4     | I4          | I3     | I2     |    |    | I1 |                      | r3 not ready     |
| 5     | I5          | I4     | I3     | I2 |    |    | I1                   | I3 needs RR!!!   |
| 6     | I6          | I5     | I4     | I3 | I2 |    |                      |                  |
| 7     | I7          | I6     | I5, I4 | I3 | I2 |    |                      | r2, r3 not ready |
| 8     | I8          | I7     | I6, I5 | I4 | I3 |    |                      | r2 not ready     |
| 9     | I9          | I8     | I6, I5 | I7 | I4 |    |                      | r2 in use        |
| 10    | I10         | I9     | I6     | I5 | I8 | I7 |                      | r3 in use        |
| 11    | I10         | I9, I6 |        | I5 | I7 | I8 |                      | r2, r3 not ready |
| 12    | I10, I9, I6 |        |        | I7 | I5 |    | r1, r2 not ready     |                  |
| 13    | I10, I9     | I6     |        | I7 | I5 |    | I6 - I7 (r2) depend. |                  |
| 14    | I10, I9     | I6     |        | I7 |    |    |                      |                  |
| 15    | I10         | I9     |        | I6 |    |    |                      |                  |
| 16    | I10         |        |        | I9 |    |    |                      |                  |
| 17    | I10         |        |        | I9 |    |    |                      |                  |
| 18    | I10         |        |        |    |    |    |                      |                  |
| 19    | I10         |        |        |    |    |    |                      |                  |
| 20    | I10         |        |        |    |    |    |                      |                  |

# Result with two functional units

| Cycle |  IF   |  ID   |  IW   |  RR   |  EX   |  RB   |  WB   | Comments                  |
|-------|-------|-------|-------|-------|-------|-------|-------|---------------------------|
|   1   | I1,I2 |       |       |       |       |       |       |                           |
|   2   | I3,I4 | I1,I2 |       |       |       |       |       | r3 not ready              |
|   3   | I5,I6 | I3,I4 | I2    |       | I1    |       |       |                           |
|   4   | I7,I8 | I5,I6 | I2    | I3    | I4    | I1    |       |                           |
|   5   | I9,I10| I7,I8 | I5,I6 | I2    | I3    | I4    |       | r3 in use, r1,r2 not ready |
|   6   |       | I9,I10| I5,I6 | I8    | I2,I7 | I3,I4 |       | EX busy                   |
|   7   |       |       | I6,I9 | I10   | I5    | I8    | I2    | r1,r2,r3 not ready        |
|   8   |       |       | I6,I9 | I10   | I5    | I7    | I8    |                           |
|   9   |       |       | I6,I9 | I10   |       | I7    | I5    |                           |
|  10   |       |       | I9,I10| I6    |       | I7    |       |                           |
|  11   |       |       | I9,I10| I6    |       | I7    |       |                           |
|  12   |       |       | I10   | I9    | I6    |       |       |                           |
|  13   |       |       | I10   | I9    |       |       |       |                           |
|  14   |       |       | I10   |       | I9    |       |       |                           |
|  15   |       |       | I10   |       |       |       |       |                           |
|  16   |       |       | I10   |       |       |       |       |                           |
|  17   |       |       | I10   |       |       |       |       |                           |

## Instruction Set Architecture

We present a list of instructions typical of a RISC (reduced set instruction computer) machine. In data-movement and control instructions, the addresses may be immediate #X, direct (memory) M,
indirect (memory) [M], register r, or register indirect [r] addresses. Data-processing instructions (arithmetic and logic instructions) use register addressing. PC is the programme counter and
a <- b indicates that the value of b is placed in a.

| Command                                        | Meaning                                        |
|------------------------------------------------|------------------------------------------------|
| LOAD a, b                                      | a <- b                                         |
| STOR a, b                                      | a <- b                                         |
| ADD a, b, c                                    | a <- b + c                                     |
| SUB a, b, c                                    | a <- b- c                                      |
| MUL a, b, c                                    | a <- b * c                                     |
| DIV a, b, c                                    | a <- b / c                                     |
| AND a, b, c                                    | a <- b & c                                     |
| OR a, b, c                                     | a <- b | c                                     |
| NOT a, b                                       | a <- !b                                        |
| ASH a, b, c                                    | a <- b (arithmetically) shifted by c positions |
| LSH a, b, c                                    | a <- b (logically) shifted by c positions      |
| BR a                                           | PC <- a                                        |
| BEQ a, b, c                                    | PC <- a if b is equal to c                     |
| BNE a, b, c                                    | PC <- a if b is not equal to c                 |
| BLT a, b, c                                    | PC <- a if b is less than c                    |
| BGT a, b, c                                    | PC <- a if b is greater than c                 |
| BLE a, b, c                                    | PC <- a if b is less than or equal to c        |
| BGE a, b, c                                    | PC <- a if b is greater than or equal to c     |

We will use a five-stage pipeline: IF (instruction fetch), ID (instruction decode), RR (register read), EX (execute instruction), WB (write back result into register). 
Note that for some instructions (e.g., LOAD ra, #X) some of the pipeline stages (e.g., RR) are not needed. Follow the rules below if you are unclear.

 1. All instructions go through the IF and ID stages.
 2. If operands are read from a register then RR is required.
 3. If data is written to a register WB is required.
 4. We will assume that for data-movement instructions the data transfer between the CPU and main memory happens in the execute stage. (This means that if a data transfer operation is executing, no data can be transferred across the bus)
 5. For data movement instructions, if no data is moved between CPU and main memory then EX is not required (e.g. LOAD r1 #X).
 6. All arithmetic and logic instructions need RR, EX and WB
 7. Successful branching operations require RR, EX and WB unless all operands are immediate then only EX, WB.
 8. Unsuccessful branching operations require RR, EX unless all operands are immediate then only EX.
 9. You may assume that the ISA supports floating point arithmetic with no loss of precision
