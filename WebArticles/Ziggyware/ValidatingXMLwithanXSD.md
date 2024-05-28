# Validating XML with an XSD


Validating an XML File vs a schema file (XSD) is very easy to do in C#: 

Lets declare a function that takes three parameters:
string strXML - In put XML file
string strXSD - In put XSD file
string strXSDNS - Input Name Space

```csharp
void ValidateXML(string strXML,string strXSD,string strXSDNS) 

void ValidateXML(string strXML,string strXSD,string strXSDNS)
{
```


Create a XmlValidatingReader, XmlSchemaCollection and ValidationEventHandler objects to be used to validate the XML against an XSD file 

```csharp
    XmlValidatingReader reader  = null;
    XmlSchemaCollection myschema = new XmlSchemaCollection();
    ValidationEventHandler eventHandler = new ValidationEventHandler(ShowCompileErrors );
```


Load the XML Document: 

```csharp
    try
    {
        //Create the XML fragment to be parsed.

        XmlDocument doc = new XmlDocument();
        doc.Load(strXML);

        string xmlFrag = doc.InnerXml;
```        


Create an XmlParserContext object for use with the XMLValidatingReader: 

```csharp
        //Create the XmlParserContext.
        XmlParserContext context = new XmlParserContext(null, null, "", XmlSpace.None);

        //Implement the reader. 
        reader = new XmlValidatingReader(xmlFrag, XmlNodeType.Element, context);
```


Add the relevant schema files (.XSD) to the name space we are checking against and repeat until all the schema's have an associated XSD file with them. 

```csharp        
        //Add the schema.
        myschema.Add(strXSDNS, strXSD);

        //Set the schema type and add the schema to the reader.
        reader.ValidationType = ValidationType.Schema;
        reader.Schemas.Add(myschema);
```


Read in the XML Data: 

```csharp
        while (reader.Read())
        {
        }
```


If there is no exception, display "Completed Validating" or return a value indicating success 

```csharp       
        textBox1.Text = "Completed validating " + strXML;
    }
```


Watch out for exceptions within the XML file and display them appropriately to the user: 

```csharp
    catch (XmlException XmlExp)
    {
        textBox1.Text = "XMLException " + XmlExp.Message;
    }
```

XMLSchemaExceptions are thrown when the XML document does not match the schema provided. 

```csharp
    catch(XmlSchemaException XmlSchExp)
    {
        textBox1.Text = "XMLSchemaException " + XmlSchExp.Message;
    }
```


Catch all other exceptions and report them to the user: 

```csharp
    catch(Exception GenExp)
    {
        textBox1.Text = "Exception " + GenExp.Message;
    }
}
```

Here is a simple implementation of how to use the ValidateXML(..) function: 

```csharp
private void button1_Click(object sender, System.EventArgs e)
{
    ValidateXML("test.xml","test.xsd","urn:bookstore-schema");
}
```