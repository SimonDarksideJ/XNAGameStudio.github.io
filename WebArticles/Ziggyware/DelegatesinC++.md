# Delegates in C++


Delegates are function pointer containers that are used as a generic form of holding function pointers.

```cpp
    template<class T>
    class ZDelegate
    {
    public:
        List<T> functionList;
        void operator += (T h)
        {
            if(!functionList.Contains(h))
                functionList.AddItem(h); 
        }
 
        void operator -= (T h)
        {
            if(functionList.Contains(h))
               functionList.RemoveItem(h); 
        }
    };
```