# Faster Matrix Identity


By using MMX we can increase the speed of a matrix identity operation.

* **emms** - begin or end using MMX
* **movd** - move a 32 bit number into an MMX register (MMX registers are 64 bit)
* **movq** - move a 64 bit number to or from an MMX register
* **0x3F800000** - This is the 32 bit hex number that represents a floating point 1.0

```cpp
    inline Matrix4& ToIdentity()
    {
        _asm
        {
            emms
            mov esi,[this.m_val]
            mov ebx,0
            movd mm1, ebx
            movq [esi],mm1
            movq [esi+8],mm1
            movq [esi+16],mm1
            movq [esi+24],mm1
            movq [esi+32],mm1
            movq [esi+40],mm1
            movq [esi+48],mm1
            movq [esi+56],mm1
            emms
            mov ebx,0x3F800000
            mov [esi],ebx
            mov [esi+20],ebx
            mov [esi+40],ebx
            mov [esi+60],ebx
        }
        return *this;
    }
```