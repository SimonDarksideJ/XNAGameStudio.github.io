# Advanced Sockets Implementation in C#


Socket programming in C# is much easier than it is in c/c++. Here we use attributes and reflection to re-create and populate incoming packets. Check it out!

Special thanks to Krissa for this great idea!


```csharp
    // Message Stream
    // class MessageAtrribute

    [AttributeUsage(AttributeTargets.Class)]
    public class MessageAttribute : Attribute
    {
        // Member Variables
        private int m_messageID;
        

        // Constructor
        public MessageAttribute(int messageID)
        {
            m_messageID = messageID;
        }
        

        // Accessor Methods
        public int ID
        {
            get
            {
                return (m_messageID);
            }
        }

        

    }

    


    // class Message

    public class Message
    {

        // MemberVariables

        private Hashtable m_ids = new Hashtable();

        

        // AccessorMethods

        public int ID
        {
            get
            {
                return ((int)m_ids[this.GetType()]);
            }
        }

        

        // Constructor

        public Message()
        {
            if (!m_ids.ContainsKey(this.GetType()))
            {
                object[] attributes = this.GetType().GetCustomAttributes(typeof(
                    MessageAttribute), true);
                int id = 0;
                if (attributes.Length > 0)
                {
                    MessageAttribute ma = (MessageAttribute)attributes[0];
                    id = ma.ID;
                }
                m_ids.Add(this.GetType(), id);
            }
        }

        

        // virtual void Read( int size, BinaryReader reader )

        public virtual void Read(int size, BinaryReader reader)
        {
        }

        

        // virtual void Write( BinaryWriter writer )

        public virtual void Write(BinaryWriter writer)
        {
        }

        

    }

    

    // class MessageStream

    public class MessageStream
    {

        // MemberVariables

        private Socket m_networkSocket;
        private NetworkStream m_networkStream;

        private BinaryReader m_networkReader;
        private BinaryWriter m_networkWriter;

        private bool m_incomingMessageGotSize = false;
        private int m_incomingMessageIndex;
        private int m_incomingMessageID = 0;
        private int m_incomingMessageSize = 0;

        private int m_outgoingMessageID = 0;

        private static Hashtable m_messageTypes = new Hashtable();

        

        // Constructor

        static MessageStream()
        {
            Assembly currentAssembly = Assembly.GetExecutingAssembly();
            foreach (Type currentType in currentAssembly.GetTypes())
            {
                object[] attributes = currentType.GetCustomAttributes(typeof(
                    MessageAttribute), true);
                if (attributes.Length < 1)
                    continue;

                MessageAttribute ma = (MessageAttribute)attributes[0];
                m_messageTypes.Add(ma.ID, currentType);
            }
        }

        

        // MessageStream( Socket s )

        public MessageStream(Socket s)
        {
            m_networkSocket = s;
            m_networkStream = new NetworkStream(m_networkSocket,
                FileAccess.ReadWrite, false);

            m_networkReader = new BinaryReader(m_networkStream,
                System.Text.Encoding.Unicode);
            m_networkWriter = new BinaryWriter(m_networkStream,
                System.Text.Encoding.Unicode);
        }

        

        // bool IsMessageAvailable

        public bool IsMessageAvailable
        {
            get
            {
                // we've started to read a message.
                if (m_incomingMessageGotSize)
                {
                    // if there's less bytes available than we're expecting, then we don't have a complete
                    // message
                    if (m_incomingMessageSize < m_networkSocket.Available)
                        return (false);

                    // if there's enough or more, then we do have a complete message
                    return (true);
                }

                // is there enough for the packet id + size;
                if (m_networkSocket.Available < 12)
                    return (false);

                m_incomingMessageGotSize = true;
                m_incomingMessageIndex = m_networkReader.ReadInt32();
                m_incomingMessageID = m_networkReader.ReadInt32();
                m_incomingMessageSize = m_networkReader.ReadInt32();

                if (m_incomingMessageSize <= m_networkSocket.Available)
                    return (true);

                return (false);
            }
        }

        

        // Message ReceiveMessage()

        public Message ReceiveMessage()
        {
            // determine what type of message it is
            Type messageType = typeof(Message);
            if (m_messageTypes.ContainsKey(m_incomingMessageID))
                messageType = (Type)m_messageTypes[m_incomingMessageID];

            // create the type
            Message message = (Message)messageType.Assembly.CreateInstance(
                messageType.FullName);

            // read message
            message.Read(m_incomingMessageSize, m_networkReader);

            // allow us to read the next message
            m_incomingMessageGotSize = false;

            return (message);
        }

        

        // void SendMessage( Message m )

        public void SendMessage(Message m)
        {
            MemoryStream ms = new MemoryStream();
            BinaryWriter b = new BinaryWriter(ms, System.Text.Encoding.Unicode);
            m.Write(b);

            m_networkWriter.Write(m_outgoingMessageID++);
            m_networkWriter.Write(m.ID);
            m_networkWriter.Write((int)ms.Length);
            byte[] outputBuffer = new byte[ms.Length];
            ms.Position = 0;
            ms.Read(outputBuffer, 0, (int)ms.Length);
            m_networkWriter.Write(outputBuffer);
        }

        

    }

    

    

    // Messages

    // MessageEnum

    public class MessageEnum
    {
        public const int PingPongMessage_id = 1;
        public const int TextMessage_id = 2;
    }

    

    // PingPongMessage

    [Message(MessageEnum.PingPongMessage_id)]
    public class PingPongMessage : Message
    {

        // Constructor

        public PingPongMessage()
        {
        }

        

        // TimeStamp

        private long m_timeStamp = 0;
        public long TimeStamp
        {
            get
            {
                return (m_timeStamp);
            }
            set
            {
                m_timeStamp = value;
            }
        }

        

        // void Read( int size, BinaryReader reader )

        public override void Read(int size, BinaryReader reader)
        {
            m_timeStamp = reader.ReadInt64();
        }

        

        // void Write( BinaryWriter writer )

        public override void Write(BinaryWriter writer)
        {
            writer.Write(m_timeStamp);
        }

        

    }

    

    // TextMessage

    [Message(MessageEnum.TextMessage_id)]
    public class TextMessage : Message
    {

        // Text

        private string m_Text = "";
        public string Text
        {
            get
            {
                return (m_Text);
            }
            set
            {
                m_Text = value;
            }
        }

        

        // void Read( int size, BinaryReader reader )

        public override void Read(int size, BinaryReader reader)
        {
            m_Text = reader.ReadString();
        }

        

        // void Write( BinaryWriter writer )

        public override void Write(BinaryWriter writer)
        {
            writer.Write(m_Text);
        }

        

    }

    

    

    // ZSocket

    public class ZSocket
    {
        // Member Variables

        private Socket m_sock = null;
        private MessageStream m_Stream = null;
        private ArrayList m_IncomingMessageList = null;
        private ArrayList m_OutgoingMessageList = null;

        

        // Constructor

        public ZSocket()
        {
        }

        

        // Accessor Methods
        public ArrayList IncomingMessageList
        {
            get
            {
                if (m_IncomingMessageList == null)
                {
                    m_IncomingMessageList = new ArrayList();
                }
                return m_IncomingMessageList;
            }
        }

        public ArrayList OutgoingMessageList
        {
            get
            {
                if (m_OutgoingMessageList == null)
                {
                    m_OutgoingMessageList = new ArrayList();
                }
                return m_OutgoingMessageList;
            }
        }

        public MessageStream Stream
        {
            get
            {
                if ((m_Stream == null) && (m_sock != null))
                {
                    if (m_sock.Connected)
                        m_Stream = new MessageStream(m_sock);
                }
                return m_Stream;
            }
        }

        public Socket Socket
        {
            get { return m_sock; }
            set { m_sock = value; }
        }
        

        // bool CreateSocket()
        public bool CreateSocket()
        {
            bool retVal = true;
            try
            {
                m_sock = new System.Net.Sockets.Socket(AddressFamily.InterNetwork,
                    SocketType.Stream,
                    ProtocolType.Tcp);
            }
            catch (Exception)
            {
                retVal = false;
            }
            return retVal;
        }
        

        // bool Connect(string host,int port)
        public bool Connect(string host, int port)
        {
            bool retVal = true;
            try
            {
                IPHostEntry IPHost = Dns.Resolve(host);

                string[] aliases = IPHost.Aliases;
                IPAddress[] addr = IPHost.AddressList;
                EndPoint ep = new IPEndPoint(addr[0], port);
                m_sock.Connect(ep);
                retVal = m_sock.Connected;
            }
            catch (Exception)
            {
                retVal = false;
            }
            return retVal;
        }
        

        // void Disconnect()
        public void Disconnect()
        {
            if (m_sock != null)
            {
                try
                {
                    IncomingMessageList.Clear();
                    OutgoingMessageList.Clear();

                    m_sock.Shutdown(SocketShutdown.Both);
                    m_sock.Close();
                }
                catch (Exception) { }
            }
        }
        

        // bool Receive()
        public bool Receive()
        {
            bool bRet = false;

            if (m_sock == null)
                return bRet;

            if (!m_sock.Connected)
                return bRet;

            while (this.Stream.IsMessageAvailable)
            {
                IncomingMessageList.Add(Stream.ReceiveMessage());
                bRet = true;
            }
            return bRet;
        }
        

        // void Send()
        public void Send()
        {
            if (m_sock == null)
                return;

            if (!m_sock.Connected)
                return;

            foreach (Message m in OutgoingMessageList)
            {
                Stream.SendMessage(m);
            }
            OutgoingMessageList.Clear();
        }
        
    }
    

    // ServerManager
    public class ServerManager
    {
        // MemberVariables
        ZSocket m_ListeningSocket = null;
        int m_ListeningPort = 0;
        private bool m_bLocalOnly = false;
        ArrayList m_ConnectedSockets = null;
        ArrayList m_ActiveSockets = null;
        

        // Accessor Methods
        // ArrayList[ZSocket] ConnectedSockets
        public ArrayList ConnectedSockets
        {
            get
            {
                if (m_ConnectedSockets == null)
                {
                    m_ConnectedSockets = new ArrayList();
                }
                return m_ConnectedSockets;
            }
        }
        

        // ArrayList[ZSocket] ActiveSockets
        public ArrayList ActiveSockets
        {
            get
            {
                if (m_ActiveSockets == null)
                {
                    m_ActiveSockets = new ArrayList();
                }
                return m_ActiveSockets;
            }
        }
        

        // Socket
        public ZSocket Socket
        {
            get { return m_ListeningSocket; }
        }
        

        

        // bool PollSockets()
        public bool PollSockets()
        {
            ActiveSockets.Clear();
            foreach (ZSocket s in ConnectedSockets)
            {
                if (s.Receive())
                {
                    ActiveSockets.Add(s);
                }
            }
            return (ActiveSockets.Count > 0);
        }
        

        // void StopServer()
        public void StopServer()
        {

            this.Socket.Disconnect();
            this.Socket.Socket = null;
        }
        

        // void StartServer(int port,bool LocalOnly)
        public void StartServer(int port, bool LocalOnly)
        {
            m_ListeningPort = port;
            m_bLocalOnly = LocalOnly;

            try
            {

                m_ListeningSocket = new ZSocket();
                m_ListeningSocket.CreateSocket();

                if (m_bLocalOnly)
                {
                    m_ListeningSocket.Socket.Bind(new IPEndPoint(IPAddress.Loopback, m_ListeningPort));
                }
                else
                {

                    IPAddress[] aryLocalAddr = null;
                    String strHostName = "";
                    try
                    {
                        // NOTE: DNS lookups are nice and all but quite time consuming.
                        strHostName = Dns.GetHostName();
                        IPHostEntry ipEntry = Dns.GetHostByName(strHostName);
                        aryLocalAddr = ipEntry.AddressList;
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine("Error trying to get local address {0} ", ex.Message);
                    }

                    // Verify we got an IP address. Tell the user if we did
                    if (aryLocalAddr == null %7C%7C aryLocalAddr.Length < 1)
                    {
                        Console.WriteLine("Unable to get local address");
                        return;
                    }

                    m_ListeningSocket.Socket.Bind(new IPEndPoint(aryLocalAddr[0], m_ListeningPort));
                }

                m_ListeningSocket.Socket.Listen(10);

                // Setup a callback to be notified of connection requests
                m_ListeningSocket.Socket.BeginAccept(
                           new AsyncCallback(OnConnectRequest), m_ListeningSocket);
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine(ex);
            }
        }
        

        // void OnConnectRequest( IAsyncResult ar )
        private void OnConnectRequest(IAsyncResult ar)
        {
            ZSocket listener = (ZSocket)ar.AsyncState;
            NewConnection(listener.Socket.EndAccept(ar));
            listener.Socket.BeginAccept(new AsyncCallback(OnConnectRequest), listener);
        }
        

        // void NewConnection( Socket sockClient )
        private void NewConnection(Socket sockClient)
        {
            ZSocket zs = new ZSocket();
            zs.Socket = sockClient;
            ConnectedSockets.Add(zs);
        }
        
    }
    ```