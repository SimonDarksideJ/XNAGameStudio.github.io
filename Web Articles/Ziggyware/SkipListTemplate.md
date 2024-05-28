# Skip List Template


Skip lists are used to store large amounts of data in a sorted collection of lists. There is a very low cost for inserting and removing nodes from the skip list.

[Here](http://www.iam.unibe.ch/~wenger/DA/SkipList/) is a great java applet demonstrating a skip lists functionality. 

```cpp
    template 
    class  ZSkipList
    {
    public:
        template 
        struct  SLNode 
        {
        public:
            T Data;                     /* user's data */
            SLNode *Forward[1];   /* skip list forward pointer */
        };

        DWORD Depth;
    private:

        SLNode *NIL;                  /* invalid node ptr */
        SLNode *Hdr;                  /* List Header */
        SLNode** update; //temporary update list
        int ListLevel;              /* current level of list */
        int maxLevel;

    public:

        DWORD GetNumNodes(){ return Depth;}

        SLNode* Begin()
        {
            return Hdr;
        }

        class  Iterator
        {
            SLNode* hdr;
            SLNode* current;
        public:

            Iterator(SLNode* hdr)
            {
                this->hdr = hdr;
                this->current = this->hdr;
            }

            Iterator& operator =(SLNode* hdr)
            {
                this->hdr = hdr;
                this->current = this->hdr;
                return (*this);
            }

            bool Next()
            {
                return ((current=current->Forward[0]) != hdr);
            }

            operator T&(){return current->Data;}
        };

        

        ~ZSkipList()
        {
            delete[] update;
            Clear();
            delete[] this->Hdr;
        }

        /**************************
        *  initialize skip list  *
        **************************/
        ZSkipList(int maxLevel= 10) 
        {
            Depth=0;

            this->maxLevel = maxLevel;
            
            if ((this->Hdr = new SLNode[this->maxLevel+1]) == 0) 
            {
                printf ("insufficient memory (InitList)\n");
                exit(1);
            }

            this->NIL = this->Hdr;

            for (int i = 0; i <= maxLevel; i++)
            {
                this->Hdr->Forward[i] = NIL;
            }

            this->ListLevel = 0;

            this->update = new SLNode*[this->maxLevel+1];
        }

        SLNode *InsertNode(T Data) 
        {
            int i;
            /***********************************************
            *  allocate node for Data and insert in list  *
            ***********************************************/

            /* find where data belongs */
            SLNode *X = this->Hdr;
            for (i = this->ListLevel; i >= 0; i--) 
            {
                while (X->Forward[i] != NIL && (X->Forward[i]->Data < Data))
                {
                    X = X->Forward[i];
                }
                update[i] = X;
            }
            
            X = X->Forward[0];
            
            if(!bAllowDuplicates)
                if (X != NIL && (X->Data == Data)) 
                {
                    return X;
                }

            /* determine level */
            int NewLevel = 0;

            while (rand() < RAND_MAX/2)
            {
                NewLevel++;
            }
            if (NewLevel > this->maxLevel) 
                NewLevel = this->maxLevel;

            if (NewLevel > this->ListLevel) 
            {
                for (i = this->ListLevel + 1; i <= NewLevel; i++)
                {
                    update[i] = NIL;
                }
                this->ListLevel = NewLevel;
            }

            /* make new node */
            if ((X = new SLNode[NewLevel+1]) == 0) 
            {
                printf ("insufficient memory (InsertNode)\n");
                exit(1);
            }
            X->Data = Data;

            /* update forward links */
            for (i = 0; i <= NewLevel; i++) 
            {
                X->Forward[i] = update[i]->Forward[i];
                update[i]->Forward[i] = X;
            }

            Depth++;

            return X;
        }

        void DeleteNode(T Data) 
        {
            int i;
            /*******************************************
            *  delete node containing Data from list  *
            *******************************************/

            /* find where data belongs */
            SLNode *X = this->Hdr;
            for (i = this->ListLevel; i >= 0; i--) 
            {
                while (X->Forward[i] != NIL && (X->Forward[i]->Data < Data))
                {
                    X = X->Forward[i];
                }
                update[i] = X;
            }

            X = X->Forward[0];
            
            if (X == NIL || !(X->Data == Data))
            {
                return;
            }

            Depth--;
            /* adjust forward pointers */
            for (i = 0; i <= this->ListLevel; i++) 
            {
                if (update[i]->Forward[i] != X)
                {
                    break;
                }
                update[i]->Forward[i] = X->Forward[i];
            }

            delete[] X;

            /* adjust header level */
            while ((this->ListLevel > 0) && (this->Hdr->Forward[this->ListLevel] == NIL))
            {
                this->ListLevel--;
            }

        }

        SLNode *FindNode(T Data) 
        {
            /*******************************
            *  find node containing Data  *
            *******************************/
            SLNode *X = this->Hdr;
            for (int i = this->ListLevel; i >= 0; i--) 
            {
                while (X->Forward[i] != NIL && (X->Forward[i]->Data < Data))
                {
                    X = X->Forward[i];
                }
            }
            
            X = X->Forward[0];
            
            if (X != NIL && (X->Data == Data))
            {
                return (X);
            }

            return NULL;
        }


        void Clear()
        {
            Depth=0;
            SLNode *Next=NIL;
            SLNode *X = this->Hdr->Forward[0];

            while(X!=NIL)
            {
                Next=X->Forward[0];
                delete[] X;
                X=Next;
            }
            
            for(int x=0;x<=this->maxLevel;x++)
                this->Hdr->Forward[x] = NIL;

            ListLevel=0;
        }
    };
```