# Queue template class


A queue is a first in - last out container. 

```cpp
    template 
    class  ZQueue
    {
    public:
        class  QueueNode
        {
            friend class ZQueue;

        public:

            T m_Data;
            QueueNode* m_pNext;
        public:

            inline operator T&()
            {
                return m_Data;
            }

            QueueNode(T pData) : m_pNext(0) { m_Data = pData; }
            QueueNode() : m_pNext(0){}
        };

        class  Iterator
        {
            QueueNode* m_pCurrent;
            bool m_bFirst;
        public:
            Iterator(QueueNode* pBegin) : m_pCurrent(pBegin), m_bFirst(true) {}

            operator T&(){ return m_pCurrent->m_Data;}

            QueueNode* Next()
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
        QueueNode* m_pHead;
        QueueNode* m_pEnd;
    public:

        ~ZQueue()
        {
            Clear();
        }

        void Clear()
        {
            QueueNode* pNode = m_pHead;
            while(pNode)
            {
                QueueNode* pNext = pNode->m_pNext;
                delete pNode;
                pNode = pNext;
            }
            m_pHead=0;
            m_pEnd=0;
        }

        ZQueue() : m_pHead(0), m_pEnd(0)
        {
        }

        QueueNode* End()
        {
            return m_pEnd;
        }

        QueueNode* Begin()
        {
            return m_pHead;
        }


        void push_back(T Item)
        {
            if(m_pEnd)
            {
                m_pEnd->m_pNext = new QueueNode(Item);
                m_pEnd = m_pEnd->m_pNext;
            }
            else
            {
                m_pHead = new QueueNode(Item);
                m_pEnd = m_pHead;
            }
        }
    };
```