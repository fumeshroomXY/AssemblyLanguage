# Directives
In RX assembly, a word starting with a dot like `.DEFINE`, `.EQU`, `.define`, `.glb`, `.section`, `.stack` is a directive telling the assembler what to do.

## Directive 1
```markdown
STACKVALSIZE    .DEFINE 0       ; Stack variant size by byte
PUSHPOPS        .DEFINE 0       ; registers
KEEPREGISTERS   .DEFINE 4*4
STACKSIZE       .EQU    STACKVALSIZE + KEEPREGISTERS    ; Used stack size by byte (StackValSize + Taking shelterd Reister size)
```

- `STACKVALSIZE .DEFINE 0`

  - Defines a constant named `STACKVALSIZE` with value `0`
  - This means the function has **no extra stack-allocated local variable area**
  - For example: ``` int func() {
          int x = 5;
          ...
      }```

- `PUSHPOPS .DEFINE 0`

  - Defines a constant named `PUSHPOPS` with value `0`
  - This means no extra registers are being **pushed** onto the stack for this function
 
- `KEEPREGISTERS .DEFINE 4*4`

  - Defines a constant equal to 16
  - 4*4 means “save 4 registers, each 4 bytes” (like `r6-r9`, which are 4 registers).
  - This function(callee) preserves them on the stack and restore them on exit for the caller.
  - The exact preserved set is defined by the project’s RX calling convention.
```markdown
Higher addresses
│
│  arguments (if any on stack)
│
│  return address        ← pushed by call
│  saved RBP             ← pushed by callee, "RBP" points to here
│  --------------------
│  preserved registers   ← preserved by callee
│  local variables
│  temporaries
│
└── RSP (stack grows down)
Lower
```

- `STACKSIZE .EQU STACKVALSIZE + KEEPREGISTERS`

  - Computes total stack size: 0 + 16 = 16. So `STACKSIZE` is 16 bytes

## Directive 2
```markdown
__FUNCNAME      .define _R_DSP_MatrixSub_i32i32_asm
    .section    P, code
    .glb        __FUNCNAME
    .stack      __FUNCNAME = 4 + STACKSIZE + PUSHPOPS*4
```

- `__FUNCNAME .define _R_DSP_MatrixSub_i32i32_asm`
  - Creates a symbol named `__FUNCNAME` pointing to the actual function name: `_R_DSP_MatrixSub_i32i32_asm`. In other words, the assembler can refer to the function via `__FUNCNAME` or the full name

- `.section P, code`
  - Places this code into the program/code section of memory
 

- `.glb __FUNCNAME`

  - Makes the function symbol globally visible so other files can call it
 

- `.stack __FUNCNAME = 4 + STACKSIZE + PUSHPOPS*4`

  - This sets the stack frame size for this function
    Calculation:
    - 4 = return address / call frame overhead
    - `STACKSIZE` = 16
    - `PUSHPOPS*4` = 0
    - Total = 20 bytes
    - So the assembler is telling the linker/runtime that this function uses a 20-byte stack frame

# Macro functions   
```markdown
;-----------------------------------------------------------------------------
; MACRO FUNCTIONS
;-----------------------------------------------------------------------------
_OVERFLOW_CHECK .MACRO rsltcode
.IF R_DSP_OVERFLOW_CHECK==1
    .LOCAL  not_overflow
    bno     not_overflow
    mov.l   #1, rsltcode
    not_overflow:
.ENDIF
.ENDM
;-----------------------------------------------------------------------------
; END OF MACRO FUNCTIONS
;-----------------------------------------------------------------------------
```
- `_OVERFLOW_CHECK .MACRO rsltcode`
  - Declares a macro named `_OVERFLOW_CHECK`
  - It takes one argument: `rsltcode`
  - So when the code later writes: `_OVERFLOW_CHECK r14`, the assembler substitutes the macro body, using `r14` as `rsltcode`

- `.IF R_DSP_OVERFLOW_CHECK==1`
  - It checks whether the symbol `R_DSP_OVERFLOW_CHECK` is equal to `1`
  - If yes, the block inside `.IF ... .ENDIF` is compiled into the final code
  - If no, that block is ignored


- `.LOCAL not_overflow`
  - Creates a local label(like a local variable) for this macro expansion
 
- `bno not_overflow`
  - `bno` means “branch if no overflow”
  - It checks the CPU overflow flag
  - If no overflow happened, jump to `not_overflow`
 
- `mov.l #1, rsltcode`
  - If overflow did happen, this instruction writes `1` into the register passed as the macro argument

- `not_overflow:`
  - Label target for the branch when no overflow occurs

- `.ENDM`
  - End of the macro definition
