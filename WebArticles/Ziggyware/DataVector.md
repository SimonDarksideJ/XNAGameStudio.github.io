# Data Vector


Here is a simple data vector template that can be used to store a chunk of memory:

```cpp 
    template <class T>
    class  ZVector
    {
    protected:
    public:
        DWORD m_Size;
        T* m_Stack;
        DWORD numItems;
 
        void Fill(T* pData,DWORD dwNumItems)
        {
            Grow(dwNumItems);
            memcpy(m_Stack,pData,sizeof(T)*dwNumItems);
            numItems = dwNumItems;
        }
 
        DWORD GetBufferSizeInBytes(){ return m_Size*sizeof(T); }
 
        void ByteAlignBuffer(int iByteAlignment)
        {
            int iSize = size()*sizeof(T) + iByteAlignment-((size()*sizeof(T))%iByteAlignment);
            Grow(iSize / sizeof(T));
        }
 
        inline ZVector()
        {
            m_Stack=0;
            m_Size=0;
            Grow(0);
            numItems=0;
        }
 
        inline ZVector(void* pData,DWORD dwNumElements)
        {
            m_Stack=0;
            m_Size=0;
            Grow(dwNumElements);
            memcpy(m_Stack,pData,dwNumElements*sizeof(T));
            numItems = dwNumElements;
        }
 
        virtual ~ZVector()
        {
            clear();
            Grow(0);
        }
 
        inline void CopyTo(ZVector<T> &v)
        {
            v.Grow(m_Size);
            v.clear();
            memset(v.m_Stack,0,sizeof(T)*m_Size);
 
            for(DWORD x=0;x<size();x++)
            {
                v[x] = GetAt(x);
            }
            v.numItems = numItems;
        }
 
        inline void MemCopyTo(ZVector<T> &v)
        {
            v.Grow(m_Size);
            v.clear();
            memset(v.m_Stack,0,sizeof(T)*m_Size);
 
            memcpy(v.m_Stack,m_Stack,sizeof(T)*m_Size);
            v.numItems = numItems;
        }
 
        void CopyElements(ZVector<T>&v,DWORD dwNumElements=-1L)
        {
            if(dwNumElements == -1L)
            {
                (*this) = v;
            }
            else
            {
                if(dwNumElements > v.size())
                {
                    dwNumElements = v.size();
                }
                Grow(dwNumElements);
                memcpy(begin(),v.begin(),sizeof(T)*dwNumElements);
                numItems = dwNumElements;
            }
        }
 
        void Append(ZVector<T>& v,DWORD dwLength=-1L)
        {
            if(dwLength==-1L)
            {
                dwLength = v.size();
            }
            if(dwLength > v.size())
                dwLength = v.size();
 
            Grow(size()+dwLength);
            memcpy(begin()+size(),v.begin(),dwLength*sizeof(T));
            numItems += dwLength;
        }
 
        void DropLeftData(DWORD dwNumItems)
        {
            if(dwNumItems > numItems)
                dwNumItems = numItems;
            memmove(begin(),begin()+dwNumItems,(numItems-dwNumItems)*sizeof(T));
            numItems -= dwNumItems;
        }
 
        inline void Grow(DWORD lSize, bool bZero = false)
        {
            if(m_Size)
            {
                if(lSize > m_Size)
                {
                    T* pLastStack = m_Stack;
                    m_Stack = (T*)renew(m_Stack,sizeof(T)*lSize);
 
                    if(((void*)m_Stack)==0)
                    {
                        m_Stack = (T*)znew(sizeof(T)*lSize);
 
                        if(!m_Stack)
                        {
                            ZString::OutputDebugLine("Could not reallocate: %d",lSize);
                            _asm{int 3}
                        }
 
                        memcpy(m_Stack,pLastStack,sizeof(T)*m_Size);
                        delete(pLastStack);
                    }
 
                    if(bZero){
                        memset(&m_Stack[m_Size],0,(lSize - m_Size) * sizeof(T));
                    }
                    m_Size = lSize;
                }
                else if(lSize==0 && m_Size>0)
                {
                    zdelete(m_Stack);
                    m_Stack = 0;
                    m_Size=0;
                }
            }
            else
            {
                if(lSize)
                {
                    m_Stack = (T*)znew(sizeof(T)*lSize);
                    if(((void*)m_Stack)==0)
                    {
                        ZString::OutputDebugLine("Could not allocate: %d",lSize);
                        _asm{int 3}
                    }
 
                    if(bZero)
                    {
                        memset(m_Stack,0,lSize * sizeof(T));
                    }
                }else
                {
                    m_Stack=0;
                }
                m_Size = lSize;
            }
        }
 
 
 
        inline void clear()
        {
            numItems=0;
        }
 
 
        inline T* begin()
        {
            return (m_Stack);
        }
 
 
        inline T* end()
        {
            return (m_Stack+(numItems-1));
        }
 
 
        void compact()
        {
            if(numItems < m_Size && numItems > 0)
            {
                void* pNewStack = new T[numItems];
                memcpy(pNewStack,m_Stack,sizeof(T)*numItems);
 
                delete(m_Stack);
 
                m_Size = numItems;
                m_Stack=(T*)pNewStack;
            }
            else
            {
                Grow(numItems);
            }
        }
 
        void push_back(T value)
        {
            if(m_Size < numItems+1)
            {
                Grow(m_Size + m_Size * .5f + 1,true);
            }
            T* pItem = (m_Stack + numItems);
            *pItem = value;
            numItems++;
        }
 
        void insert(DWORD i,T value)
        {
            if(m_Size < numItems+1)
            {
                Grow(m_Size + m_Size * .5f + 1);
            }
 
            //for(int x=numItems-1;x>=i;x--)
            {
                memmove(&m_Stack[i+1],&m_Stack[i],sizeof(T)*(numItems-i));
            }
            ZeroMemory(&m_Stack[i],sizeof(T));
            m_Stack[i] = value;
            numItems++;
        }
 
        inline DWORD size()
        {
            return numItems;
        }
 
 
        inline void pop_back()
        {
            if(numItems)
                numItems--;
        }
 
        inline void erase(T Item)
        {
            DWORD iNum = numItems;
            while(iNum--)
            {
                if( *(m_Stack+iNum) == Item)
                {
                    erase_at(iNum);
                    break;
                }
            }
        }
 
        inline void erase_at(DWORD offset,DWORD dwLen=1)
        {
            memcpy_erase(offset,dwLen);
        }
 
        inline void copy_erase(DWORD offset)
        {
 
            numItems--;
            if(numItems>0)
            {
                for(DWORD x=offset+1;x <= numItems;x++)
                {
                    m_Stack[x-1] = m_Stack[x];
                }
            }
 
        }
 
 
        inline void memcpy_erase(DWORD offset,DWORD dwLen=1)
        {
            if(numItems)
            {
                numItems--;
                if(numItems>0)
                {
                    if(numItems-offset)
                        memmove(&m_Stack[offset],&m_Stack[offset+dwLen],sizeof(T)*(numItems-offset));
                }
            }
        }
 
        inline T& GetAt(DWORD i)
        {
            return *(m_Stack+i);
        }
 
        inline T& operator[](DWORD a)
        {
            return GetAt(a);
        }
    };
```