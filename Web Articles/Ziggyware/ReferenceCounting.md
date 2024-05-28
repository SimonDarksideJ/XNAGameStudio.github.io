# Reference Counting

Here is an implementation of a reference counting system: 

```cpp
class MyObject : public ReferenceCounted{};
class MyObjectPtr : public IReferenceImpl<MyObject>{};
MyObjectPtr p;
p.Initialize(); // this calls the object constructor
p.Release(); //this destroys the object
```

Here is the definition of the reference counting template: 

```cpp
    struct  ReferenceCounted
    {
 
        int m_iRefCount;
    public:
 
        ReferenceCounted() : m_iRefCount(0) {}
 
        void AddRef()
        {
            ++m_iRefCount;
        }
        int Release()
        {
            return --m_iRefCount;
        }
    };
 
 
 
    template <class T>
    class  IReferenceImpl
    {
    protected:
        T* m_pData;
    private:
        void InitData(){ m_pData = 0;}
 
    public:
        IReferenceImpl()
        {
            InitData();
        }
        IReferenceImpl(T* p)
        {
            InitData();
            m_pData = p;
            if(m_pData)
                m_pData->AddRef();
        }
 
        void SetRefSource(T* p)
        {
            Release();
            m_pData = p;
            if(m_pData)
                m_pData->AddRef();
        }
 
        void SetRefSourceNoAddRef(T* p)
        {
            Release();
            m_pData = p;
        }
 
        operator T*()
        {
            return m_pData;
        }
 
        bool operator == (IReferenceImpl<T>& p)
        {
            return m_pData == p.m_pData;
        }
 
        bool operator != (IReferenceImpl<T>& p)
        {
            return m_pData != p.m_pData;
        }
 
        operator bool()
        {
            return m_pData!=0;
        }
 
        bool operator !()
        {
            return m_pData==0;
        }
 
 
 
        T* operator ->()
        {
            return m_pData;
        }
 
        virtual ~IReferenceImpl()
        {
            Release();
        }
 
        IReferenceImpl(IReferenceImpl<T>& b)
        {
            InitData();
            (*this) = b;
        }
 
        IReferenceImpl<T>& operator =(IReferenceImpl<T>& b)
        {
            Release();
            m_pData = b.m_pData;
            if(m_pData)
            {
                m_pData->AddRef();
            }
            return (*this);
        }
 
        IReferenceImpl<T>& operator =(T* b)
        {
            Release();
            m_pData = b;
            if(m_pData)
            {
                m_pData->AddRef();
            }
            return (*this);
        }
 
        void Initialize()
        {
            Release();
 
            m_pData = new T();
            m_pData->AddRef();
        }
 
        void Release()
        {
            if(m_pData)
            {
                if(m_pData->Release()==0)
                {
                    delete m_pData;
                }
                m_pData=0;
            }
        }
    };
```