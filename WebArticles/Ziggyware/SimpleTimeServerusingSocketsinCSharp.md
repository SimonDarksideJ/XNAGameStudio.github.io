# Simple Time Server using Sockets in C#


Here is a basic implementation of a socket client and server

You should include the System.Net and System.Net.Sockets and System.Text assemblies 

```csharp
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;

class Server 
{
```

Declare the main entry point for the server assembly:

```csharp
    public static void Main() 
    {
```

Create a listening socket: 

```csharp            
        try
        {

            // listen on port 14
           // Dns.Resolve() can return multiple addresses for the given IP. Lets go with the first one (index 0)

            TcpListener tcpl = new TcpListener(Dns.Resolve("localhost").AddressList[0], 14); 

            //start the listener
            tcpl.Start();
        
            Console.WriteLine("Waiting for clients to connect");
            Console.WriteLine("Press Ctrl+c to Quit...");
```

Wait for incoming connections: 

```csharp
            while (true)
            {
                // Accept will block until someone connects
                Socket s = tcpl.AcceptSocket();
```


We have a connection! Now pass the current date and time to the client.
First thing is we need to get the current time: 

```csharp
                // Get the current date and time then concatenate it
                // into a string
                DateTime now = DateTime.Now;
                String strDateLine = now.ToShortDateString() + " " + now.ToLongTimeString();
```


Convert the time into ASCII bytes to send thru the socket (.NET stores all strings as UNICODE) 

```csharp
                // Convert the string to a Byte Array and send it
                Byte[] byteDateLine = Encoding.ASCII.GetBytes(strDateLine.ToCharArray());
```               


Send the time over the socket to the client: 

```csharp
                s.Send(byteDateLine, byteDateLine.Length, 0);
```

Close the socket. 

```csharp
                s.Close();
```                


We can use Console.WriteLine to show what time the server sent to the client: 

```csharp
                Console.WriteLine("Sent {0}", strDateLine);
            }
        }
```

You should catch exceptions of type SocketException to see if there was an error while sending to the client: 

```csharp

        catch (SocketException socketError)
        {
```

This can be a common problem when you forget to close your socket before starting the socket server up again :) 

```csharp
            if (socketError.ErrorCode == 10048)
            {
                Console.WriteLine("Connection to this port failed.  There is another server is listening on this port.");
            }
        }
    }
}
```


Thats it for the time server! Lets take a look at the time client: 


```csharp

#region SocketClient
using System;
using System.Net;
using System.Net.Sockets;
using System.IO;
using System.Text;

class Client 
{
```

declare the Main entry point for the time client program: 

```csharp
    public static void Main(String[] args) 
    {
```

Create a socket to connect to the server with: 

```csharp

        TcpClient tcpc = new TcpClient();
```


Allocate a buffer to retrieve the time from the server: 


```csharp
        tcpc.ReceiveBufferSize = 32;

```


We are using a command line option that contains the server ip to connect to: 

```csharp
        if (args.Length != 1) 
        {
            Console.WriteLine("Please specify a server name in the command line");
            return;
        }
```


Get the server ip or name from the command line argument: 

```csharp
        String server = args[0];
```


Verify that the server that was passed in as the command line parameter is a valid server: 

```csharp
        // Verify that the server exists
        try 
        {
            IPHostEntry ipInfo = Dns.GetHostByName(server);
        }
```

If this is not a valid server, and exception will be thrown. 

```csharp
        catch (SocketException socketExcep) 
        {
```


Display an appropriate message to the user and exit the application: 

```csharp
            Console.WriteLine("Cannot find server: " + server + "\nException:\n" + socketExcep.ToString());
            return;
        }
```


Attempt to connect to the time server: 


```csharp
        // Try to connect to the server
        try
        {
            tcpc.Connect(server, 14);
        }
        catch(Exception e)
        {
            Console.WriteLine("Could not connect to server {0}\n" + e.Message,server);
        }
```


We use a Stream object to stream data from the server's socket: 


```csharp
        // Get the stream
        Stream s;
        try
        {
            s = tcpc.GetStream();
        }
        catch (InvalidOperationException)
        {
            Console.WriteLine("Cannot connect to server {0}", server);
            return;
        }
```


Read the ASCII data sent from the server: 


```csharp
        Byte[] read = new Byte[32];

        // Read the stream and convert it to ASII
        int bytes = s.Read(read, 0, read.Length);
        String Time = Encoding.ASCII.GetString(read);
```


Display the result to the user: 


```csharp
        Console.WriteLine("Received " + bytes + " bytes");
        Console.WriteLine("Current date and time is: " + Time);
```


Don't forget to close the client socket: 


```csharp
        tcpc.Close();
```


By calling Console.Read() we can make the window stay open when executing the program in debug mode: 

```csharp
        // Wait for user response to exit
        Console.WriteLine("Press Enter to exit");
        Console.Read();
    }
}
```