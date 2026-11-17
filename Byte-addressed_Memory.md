# Byte-addressed Memory
In most modern processors, memory is **byte-addressable**, which means:

Each memory address refers to **one byte (8 bits)** of data.

## Array Example
Consider:
```c
char a[4];
```
Memory:
```markdown
a[0] -> 0x1000
a[1] -> 0x1001
a[2] -> 0x1002
a[3] -> 0x1003
```
Since each char is 1 byte, addresses increase by 1.

But:
```c
int a[4];
```
On a 32-bit system, each int is 4 bytes:
```markdown
a[0] -> 0x1000
a[1] -> 0x1004
a[2] -> 0x1008
a[3] -> 0x100C
```
The compiler multiplies the index by 4 because each element occupies 4 byte addresses.

This is why the following code makes sense:
```c
#include <stdint.h>
void mul_8x8_16(uint8_t* a, uint8_t* b, uint16_t* c, int n) {
    for (int i=0; i<n; i++)
        c[i] = a[i]*b[i];
}
```
```markdown
; R1=a, R2=b, R3=c, R4=n
ADD     1, R1               ;increment a because every element is 1 byte
...
ADD     2, R3               ;increment c because every element is 2 bytes
```
