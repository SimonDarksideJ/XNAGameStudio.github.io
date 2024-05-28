# Linked List template class


Here is an implementation of a list class. Lists are another way to store data. Lists have very fast inserts and deletes however iterating thru the elements in the list is not as fast as iterating thru a data vector.

```cpp
    template 
    class  ZList
    {
    public:
        class ListNode;
    private:
        DWORD m_dwSize;
        bool bValid;
        
        ZVector m_Offsets;
    public:

        T AllocItem()
        {
            T ret;
            ret.Initialize();
            push_back(ret);
            return ret;
        }

        
        DWORD GetSize(){ return size(); }
        
       
        DWORD size()
        {
            return m_dwSize;
        }


        inline bool IsEmpty(){ return m_pHead==NULL;}
        

        class  ListNode
        {
            friend class ZList;
            
        public:
            

            T m_Data;
            ListNode* m_pNext;
            ListNode* m_pPrev;
        public:
            inline operator T&()
            {
                return m_Data;
            }

            ListNode(T pData) : m_pNext(0), m_pPrev(0) { m_Data = pData; }
            ListNode() : m_pNext(0), m_pPrev(0){}
        };

        class  Iterator
        {
            ListNode* m_pCurrent;
            bool m_bFirst;
        public:
            Iterator(ListNode* pBegin) : m_pCurrent(pBegin), m_bFirst(true) {}

            operator T&(){ return m_pCurrent->m_Data;}

            ListNode* Next()
            { 
                if(m_bFirst)
                { 
                    m_bFirst = false; 
                    return m_pCurrent;
                }
                else
                {
                    m_pCurrent = m_pCurrent->m_pNext;
                    return m_pCurrent;
                }
            }
        };

    private:
        ListNode* m_pHead;
        ListNode* m_pEnd;
    public:

        ~ZList()
        {
            Clear();
        }

        void Clear()
        {
            bValid  =false;
            ListNode* pNode = m_pHead;
            while(pNode)
            {
                ListNode* pNext = pNode->m_pNext;
                delete pNode;
                pNode = pNext;
                m_dwSize--;
            }
            m_pHead=0;
            m_pEnd=0;
        }

        ZList() : m_pHead(0), m_dwSize(0), m_pEnd(0), bValid(false)
        {
        }

        ListNode* End()
        {
            return m_pEnd;
        }

        ListNode* Begin()
        {
            return m_pHead;
        }

        void Add(T& Item)
        {
            push_back(Item);
        }



        void push_back(T Item)
        {
            bValid=false;

            if(m_pEnd)
            {
                m_pEnd->m_pNext = new ListNode(Item);
                m_pEnd->m_pNext->m_pPrev = m_pEnd;
                m_pEnd = m_pEnd->m_pNext;
            }
            else
            {
                m_pHead = new ListNode(Item);
                m_pEnd = m_pHead;
            }
            m_dwSize++;
        }

        void Insert(T Item,ListNode* pPrev = 0)
        {
            bValid=false;
            m_dwSize++;

            if(!pPrev)
            {
                ListNode* pBase = m_pHead;
                m_pHead = new ListNode(Item);
                m_pHead->m_pNext = pBase;
                if(!pBase)
                    pBase = m_pHead;
                pBase->m_pPrev = m_pHead;
                if(!m_pEnd)
                {
                    m_pEnd = pBase;
                }
            }
            else
            {
                ListNode* pNext = pPrev->m_pNext;
                if(!pNext)
                {
                    pPrev->m_pNext = new ListNode(Item);
                    pPrev->m_pNext->m_pPrev = pPrev;
                    if(m_pEnd == pPrev)
                    {
                        m_pEnd = pPrev->m_pNext;
                    }
                }
                else
                {
                    pPrev->m_pNext = new ListNode(Item);
                    pPrev->m_pNext->m_pPrev = pPrev;
                    pPrev->m_pNext->m_pNext = pNext;
                    pNext->m_pPrev = pPrev->m_pNext;
                    if(m_pEnd == pPrev)
                    {
                        m_pEnd = pPrev->m_pNext;
                    }
                }
            }
        }

        void RemoveItem(T Item)
        {
            Iterator i = Begin();
            while(ListNode* pNode = i.Next())
            {
                if(pNode->m_Data == Item)
                {
                    RemoveItem(pNode);
                    return;
                }
            }
        }

        void RemoveAt(DWORD dw)
        {
            if(!bValid)
            {
                m_Offsets.clear();
                Iterator i = Begin();
                while(ListNode*p = i.Next())
                {
                    m_Offsets.push_back(p);
                }
                bValid=true;
            }
            RemoveItem(m_Offsets[dw]);
        }

        ListNode* GetNodeAt(DWORD dw)
        {
            if(!bValid)
            {
                m_Offsets.clear();
                Iterator i = Begin();
                while(ListNode*p = i.Next())
                {
                    m_Offsets.push_back(p);
                }
                bValid=true;
            }
            return m_Offsets[dw];
        }

        
        

        T& operator[](DWORD dw)
        {
            if(!bValid)
            {
                m_Offsets.clear();
                Iterator i = Begin();
                while(ListNode*p = i.Next())
                {
                    m_Offsets.push_back(p);
                }
                bValid=true;
            }
            return m_Offsets[dw]->m_Data;
        }

        T& GetAt(DWORD dw)
        {
            return (*this)[dw];
        }

        void RemoveItem(ListNode* pNode)
        {
            bValid=false;
            m_dwSize--;

            ListNode* pPrevItem = pNode->m_pPrev;
            ListNode* pNextItem = pNode->m_pNext;

            if(pNode == m_pHead)
            {
                m_pHead = m_pHead->m_pNext;
            }

            if(pNode == m_pEnd)
            {
                m_pEnd = m_pEnd->m_pPrev;
            }

            delete pNode;

            if(pPrevItem)
                pPrevItem->m_pNext = pNextItem;
            if(pNextItem)
                pNextItem->m_pPrev = pPrevItem;
        }

        bool Contains(T& item)
        {
            Iterator i = Begin();
            while(ListNode* pNode = i.Next())
            {
                if(pNode->m_Data == item)
                {
                    return true;
                }
            }
            return false;
        }

        void CopyTo(ZList& p)
        {
            Iterator i = Begin();
            while(i.Next())
            {
                p.push_back(i);
            }
        }

    };
```