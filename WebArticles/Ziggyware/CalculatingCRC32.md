# Calculating CRC 32


Here is source code for calculating the CRC 32 (32 bit Cyclic Redundancy Check) value of a given 32 bit number. 

The calculation behind a CRC 32 value is beyond the scope of this sample, however if you would like to check it out, a good tutorial is on wikipedia 

```cpp
    class Crc32
    {
        static const DWORD Quotient;
        static DWORD crctab[256];
    public:
        static void Initialize();
        static DWORD Calc32(DWORD* data,DWORD Length);
    };


    const DWORD Crc32::Quotient = 0x04c11db7;
    DWORD Crc32::crctab[256];

    void Crc32::Initialize()
    {
        int i,j;

        DWORD crc;

        for (i = 0; i < 256; i++)
        {
            crc = i << 24;
            for (j = 0; j < 8; j++)
            {
                if (crc & 0x80000000)
                    crc = (crc << 1) ^ Quotient;
                else
                    crc = crc << 1;
            }
            crctab[i] = htonl(crc);
        }

    }

    DWORD Crc32::Calc32(DWORD* data,DWORD Length)
    {
        DWORD        result;
        DWORD        *p = (DWORD *)data;
        DWORD        *e = (DWORD *)(data + Length);
        
        if (Length < 4)
        {
            return 0;
        }

        result = ~*p++;
        
        while( p> 8;
            result = crctab[result & 0xff] ^ result >> 8;
            result = crctab[result & 0xff] ^ result >> 8;
            result = crctab[result & 0xff] ^ result >> 8;
            result ^= *p++;
        }
        
        return result;
    }
```