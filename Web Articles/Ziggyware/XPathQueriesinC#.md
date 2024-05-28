# XPath Queries in C#


Here is a simple tutorial on using XPath Queries in C#: 

First lets create an XmlDocument object so we can load an XML File: 

```csharp
XmlDocument xmldoc = new XmlDocument();
```


Now we can load the xml file with the XmlDocument.Load() function 

```csharp
xmldoc.Load("ZiggySaverConfig.xml");
```


Create a stream reader to extract the InnerXML from the XmlDocument we just loaded: 

```csharp
System.IO.StringReader sr = new System.IO.StringReader(xmldoc.InnerXml);
```


With this StreamReader object, we can create an XPathDocument object so we can run XPath queries 

```csharp
XPathDocument doc = new XPathDocument(sr);
```


Now we need to create an XPathNavigator object that will allow us to compile an XPath query and select values from the XmlDocument :) 

```csharp
XPathNavigator nav = doc.CreateNavigator(); 
```


The Compile method of the XPathNavigator will increase the speed at which XPath data selections can occur in the XmlDocument 

```csharp
XPathExpression expImgSize = nav.Compile(@"/ZiggySaverConfig/@ImageSize");
```


Call the Select() method of the XPathNavigator to retrieve an interator container of XPathNode elements that were returned from the XPath query 

```csharp
XPathNodeIterator iterImageSize = nav.Select(expImgSize);
```


Dont forget to check to see if the iterator is null, this would indicate an empty return set from the XPath query. 

```csharp
if(iterImageSize != null)
{
```


The XpathNodeIterator is traversed by calling the MoveNext() method. 
You must call this method before trying to retrieve any data from the iterator: 

```csharp
    while(iterImageSize.MoveNext())
    {
```


Get the value of the XPath query result by using the iterator's Current.Value property 


```csharp
        imageSize = iterImageSize.Current.Value;
    }
}
```