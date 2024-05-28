# Finite State Machines


Finite State Machines are used to keep the state of a system of objects. In game development they can be used to make your engine event driven. 

```cpp
    template<class tKey,class tKeyParam,class tData,class tDataParam>
    class FSM
    {
    public:
        typedef void (*FSMCallback)(void);
 
        typedef ZMap<tData,tDataParam,FSMCallback,FSMCallback> ZStateCallbackMap;
    private:
        ZVector<ZStateCallbackMap*> m_Callbacks;
 
        ZMap<tKey,tKeyParam,tData,tDataParam>      m_States;
        ZMap<tKey,tKeyParam,ZStateCallbackMap*,ZStateCallbackMap*>  m_StateCallbacks;
        ZMap<tKey,tKeyParam,FSMCallback,FSMCallback>         m_StateCallbacks2;
 
    public:
        void SetState(tKeyParam cpState, tDataParam dwValue)
        {
            if(!m_States.Exists(cpState))
            {
                m_States.Insert(cpState,dwValue);
            }
            else
            {
                m_States.Get(cpState) = dwValue;
            }
 
            //invoke the callback
            if(m_StateCallbacks.Exists(cpState))
            {
                ZStateCallbackMap* pCB = m_StateCallbacks.Get(cpState);
                if(pCB->Exists(dwValue))
                {
                    pCB->Get(dwValue)();
                }
            }
 
            if(m_StateCallbacks2.Exists(cpState))
            {
                m_StateCallbacks2.Get(cpState)();
            }
        }
        void SetStateChangeCallback(tKeyParam cpState,tDataParam dwValue,FSMCallback fn)
        {
            ZStateCallbackMap * pCBMap = 0;
 
            if(!m_StateCallbacks.Exists(cpState))
            {
                pCBMap = new ZStateCallbackMap();
                m_Callbacks.push_back(pCBMap);
                m_StateCallbacks.Insert(cpState,pCBMap);
            }
            else
            {
                pCBMap = m_StateCallbacks.Get(cpState);
            }
 
            if(!pCBMap->Exists(dwValue))
            {
                pCBMap->Insert(dwValue,fn);
            }
            else
            {
                pCBMap->Get(dwValue) = fn;
            }
        }
 
        void SetStateChangeCallback(tKeyParam cpState,FSMCallback fn)
        {
            if(!m_StateCallbacks2.Exists(cpState))
            {
                m_StateCallbacks2.Insert(cpState,fn);
            }
            else
            {
                m_StateCallbacks2.Get(cpState) = fn;
            }
        }
        tDataParam GetState(tKeyParam cpState)
        {
            if(m_States.Exists(cpState))
            {
                return m_States.Get(cpState);
            }
            else
            {
                ZAssert(0,"State not defined");
                return 0;
            }
        }
 
        ~FSM()
        {
            for(DWORD x=0;x<m_Callbacks.size();x++)
            {
                delete m_Callbacks[x];
            }
            m_Callbacks.clear();
        }
    };
```

Here we have a templated finite state machine that can invoke functions when a state is changed. 
This is very useful when designing an event driven system.