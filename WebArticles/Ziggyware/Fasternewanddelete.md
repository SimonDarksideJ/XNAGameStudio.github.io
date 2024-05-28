# Faster new and delete


Below is a way you can get alot more bang for your buck when using new and delete.
We use the Win32 function GetProcessHeap() to retrieve a handle to the heap and overload the new and delete operators to use HeapAlloc, HeapRealloc and HeapFree. 

```cpp
    static HANDLE hProcessHeap = GetProcessHeap();
    inline void * __cdecl operator new(size_t size)
    {	
        return (void *)HeapAlloc(hProcessHeap,0,size);
    };

    inline void * __cdecl renew(void* pData,size_t size)
    {    
        return (void *)HeapReAlloc(hProcessHeap,0,pData,size);    
    };

    inline void * __cdecl operator new[](size_t size)
    {
        return HeapAlloc(hProcessHeap,0,size);
    };

    inline void __cdecl operator delete(void *p)
    {
        HeapFree(hProcessHeap,0,p);
    };

    inline void __cdecl operator delete[](void *p)
    {
        HeapFree(hProcessHeap,0,p);
    };
```