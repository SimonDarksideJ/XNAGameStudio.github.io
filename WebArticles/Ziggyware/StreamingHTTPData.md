# Streaming HTTP Data


Downloading HTML from the internet is a breeze with C#: 

Parameters: string strURL - a url to the page you would like to download from: example: "www.google.com" 

Return Value: returns the HTML data retrieved from the remote host 

```csharp
static string GetHTMLFromURL(string strURL)
{
    string retVal = "";

    try
    {
```


Create a new HttpWebRequest object. This object is used to download information from the internet. 

```csharp
        HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(strURL);
```


call the GetResponse() method to obtain a connection to the remote site: 

```csharp
        // execute the request
        HttpWebResponse response = (HttpWebResponse)
            request.GetResponse();
```


Retrieve the data stream from the response object: 

```csharp
        // we will read data via the response stream
        Stream resStream = response.GetResponseStream();
```


Accumulate the data from the response stream: 

```csharp
        string tempString = null;
        int    count      = 0;

        StringBuilder sb = new StringBuilder();

        byte[]        buf = new byte[15000];

        do
        {
```

Fill a buffer with a segment of the stream. 

 ```csharp
            count = resStream.Read(buf, 0, buf.Length);
```


Make sure the stream was able to be read from. 

```csharp
            if (count != 0)
            {
```

Translate the byte[] data into an ASCII string 

```csharp
                tempString = Encoding.ASCII.GetString(buf, 0, count);
```


Append the results to a string buffer for optimal performance: 

```csharp
                sb.Append(tempString);
            }
        }
        while (count > 0); // any more data to read?
```


Get the data out of the string buffer and place it into the return value: 

```csharp
        retVal = sb.ToString();
    }
    catch(Exception)
    {
    }
    return retVal;
}
```

Thats all there is to it!