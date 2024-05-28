# TCP Socket Implementation


TCP sockets are the base line when learning how to interact with other machines on the internet.
There are many implementations on the internet, so here is mine :)
Socket programming can seem very tricky at first. I suggest you search google for each function that you are having trouble with and read up on how each function is used. 

```cpp
    class ZTCPSocket
    {
        SOCKET m_Socket;
        DWORD m_dwAddr;
        ZString m_strAddress;
        WORD m_Port;
        bool m_bReceiving;
    public:

        ZString GetAddressString();

        ZTCPSocket();
        bool Connect(const char* pAddress,WORD port);
        bool Recv(ZVector &strOut);
        bool Recv(ZString &strOut);

        bool Send(ZVector& strOut,bool bString=false);

        bool Send(ZString& strOut);


        void Close();

        static void InitializeSockets();

        static void ShutdownSockets();
    };



    ZString ZTCPSocket::GetAddressString(){ return m_strAddress;}

    ZTCPSocket::ZTCPSocket()
    {
        m_Socket = INVALID_SOCKET;
        m_dwAddr = 0;
        m_Port = 0;
        m_bReceiving=false;
    }
    bool ZTCPSocket::Connect(const char* pAddress,WORD port)
    {
        Close();
        m_dwAddr = inet_addr(pAddress);
        if(!m_dwAddr || m_dwAddr == -1L)
        {
            m_dwAddr = 0;
            hostent* host = gethostbyname(pAddress);
            if(host)
                m_dwAddr = ((struct in_addr *)(host->h_addr))->S_un.S_addr;
        }

        if(!m_dwAddr)
            return false;

        sockaddr_in sAddr;
        memset( &sAddr, 0, sizeof(sAddr) );

        sAddr.sin_family = AF_INET;
        sAddr.sin_addr.S_un.S_addr = m_dwAddr;

        m_Port = sAddr.sin_port = htons( port );
        m_Socket = socket(sAddr.sin_family,SOCK_STREAM,0);

        if(m_Socket == INVALID_SOCKET)
        {
            Close();
            return false;
        }

        int i = connect(m_Socket,(sockaddr*)&sAddr,sizeof(sockaddr_in));

        if(i == SOCKET_ERROR)
        {
            Close();
            return false;
        }
        else
        {
            m_strAddress = pAddress;
            return true;
        }
    }

    bool ZTCPSocket::Recv(ZVector &strOut)
    {
        strOut.clear();

        timeval timeout;
        fd_set socksRead;
        FD_ZERO(&socksRead);

        fd_set socksWrite;
        FD_ZERO(&socksWrite);

        fd_set socksError;
        FD_ZERO(&socksError);

        if(!m_bReceiving)
        {
            m_bReceiving = true;
            timeout.tv_sec = 5;
            timeout.tv_usec = 0;
        }
        else
        {
            timeout.tv_sec = 1;
            timeout.tv_usec = 0;
        }

        FD_SET(m_Socket,&socksRead);

        if(select(1,&socksRead,NULL,NULL,&timeout)>0)
        {
        }
        else
        {
            m_bReceiving = false;
            return false;
        }

        DWORD dwBytesAvailable;
        if( ioctlsocket(m_Socket,FIONREAD,&dwBytesAvailable) == SOCKET_ERROR)
        {
            dwBytesAvailable=0;
        }
        strOut.Grow(dwBytesAvailable+1);

        int i = recv(m_Socket,strOut.begin(),dwBytesAvailable,0);

        if(i > 0)
        {
            strOut.begin()[i]=0;
            strOut.numItems=i;
            return true;
        }
        else if(i == SOCKET_ERROR)
        {
            return false;
        }
        else
        {
            //zero length packet
            return false;
        }
    }

    bool ZTCPSocket::Recv(ZString &strOut)
    {
        return Recv(strOut.m_Data);
    }

    bool ZTCPSocket::Send(ZVector& strOut,bool bString/*=false*/)
    {
        int iSize = bString ?strlen(strOut.begin()) : strOut.GetBufferSizeInBytes();
        int i = send(m_Socket,strOut.begin(),iSize,0);
        return  i == iSize;
    }

    bool ZTCPSocket::Send(ZString& strOut)
    {
        return Send(strOut.m_Data,true);
    }


    void ZTCPSocket::Close()
    {
        if(m_Socket != INVALID_SOCKET)
            closesocket(m_Socket);
        m_Socket = INVALID_SOCKET;
    }

    void ZTCPSocket::InitializeSockets()
    {
        WSADATA wsaData;
        WSAStartup(MAKEWORD(2,0),&wsaData);
    }

    void ZTCPSocket::ShutdownSockets()
    {
        WSACleanup();
    }
```