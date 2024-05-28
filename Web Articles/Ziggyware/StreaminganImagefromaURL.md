# Streaming an Image from a URL


Streaming an image from the internet is extremely easy in C#

First off we need to include a set of assemblies in our project:

```csharp
using System;
using System.Drawing;
using System.Collections;
using System.Net;
using System.Web;
using System.Text;
using System.IO;
```

Function Declaration:

```csharp
Image GetImageFromURL(string strURL)
```

Parameters: 
string strURL - the url of the image to download

```csharp
Image GetImageFromURL(string strURL)
{
```

Initialize the return value

```csharp
    Image retVal = null;
```

Any errror while connecting to the server or streaming the data into an image can result in an exception! dont forget to use a try{}catch{} block!

```csharp
    try
    {
```

We use the HttpWebRequest object to connect to a web site:

```csharp
        HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(strURL);
```

Be sure to populate the time out values in the request :)

```csharp                    
        request.Timeout = 5000; // 5 seconds in milliseconds
        request.ReadWriteTimeout = 20000; // allow up to 20 seconds to elapse
```

Retrieving the stream of data from the web server is very easy:

```csharp
        // execute the request
        HttpWebResponse response = (HttpWebResponse)
            request.GetResponse();
```

Now we convert the downloaded stream into an Image!

```csharp
        retVal = Image.FromStream(streamThreadStream);
    }
```

Perform cleanup on an exception:

```csharp
    catch(Exception)
    {
        retVal = null;
    }
```

Thats it! now we return the image :)

```csharp
    return retVal;
}
```