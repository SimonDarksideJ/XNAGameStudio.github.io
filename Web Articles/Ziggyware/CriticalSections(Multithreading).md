# Critical Sections (Multithreading)


Here is a simple implementation of a locked variable that can be used between two threads without worrying about the memory being garbaged up with simultaneous writes:

Here is a simple usage:

```cpp
ZLockedValue intVariable;
   intVariable = 100;
   int i = intVariable;
```

Source Code: 

```cpp
    template <class T>
    class  ZLockedValue
    {
        ZLocker m_Locker;
        T m_Value;
    public:
        operator T()
        {
            T ret;
            m_Locker.Lock();
            ret = m_Value;
            m_Locker.UnLock();
            return ret;
        }
 
        void operator= (T value)
        {
            m_Locker.Lock();
            m_Value = value;
            m_Locker.UnLock();
        }
    };


    class  ZLocker
    {
        CRITICAL_SECTION m_CS;
        int m_iCount;
    public:
        ZLocker() : m_iCount(0) {::InitializeCriticalSection(&m_CS);}
        void Lock()
        {
            ::EnterCriticalSection(&m_CS);
            ZAssert(m_iCount++ == 1,"Error, multiple locks");
        }
        void UnLock()
        {
            ZAssert(m_iCount-- == 0,"Error, multiple unlocks");
            ::LeaveCriticalSection(&m_CS);
        }
    };
```