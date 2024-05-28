# Teaching a man to fish

Variants of the following question come up somewhat regularly on the forums:

* I want to import game data from an XML file
* I plan to do this using the Content Pipeline IntermediateSerializer
* I have my custom data type all ready to go
* But I don’t know how I should format the XML for it!

You already have the tools to answer this yourself. Here’s how:

Fire up Visual Studio and create a new Console Application project.

Right-click on the References node, and add the Microsoft.Xna.Framework.Content.Pipeline assembly.

Add using declarations for the System.Xml and Microsoft.Xna.Framework.Content.Pipeline.Serialization.Intermediate namespaces.

Add a test class with the same layout as whatever data you want to serialize, but initialized with dummy test values. For instance:

```csharp
    namespace XmlTest
    {
        class MyTest
        {
            public int elf = 23;
            public string hello = "Hello World";
        }
    }
```

Add this code to Main:

```csharp
    MyTest testData = new MyTest();

    XmlWriterSettings settings = new XmlWriterSettings();
    settings.Indent = true;

    using (XmlWriter writer = XmlWriter.Create("test.xml", settings))
    {
        IntermediateSerializer.Serialize(writer, testData, null);
    }
```

> To use this in MonoGame currently, you will need to reference the MonoGame Content Pipeline dll, which is normally located in
>
> "C:\Program Files (x86)\MSBuild\MonoGame\v3.0\Tools\MonoGame.Framework.Content.Pipeline.dll"
>
> **However, DO NOT leave this reference in a LIVE build, as it wil include the entire Content Pipeline architecture which is not needed for live projects.**

Run the program. Look in the bin\Debug folder, and open the test.xml output file. With the class shown above, this will look like:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="XmlTest.MyTest">
        <elf>23</elf>
        <hello>Hello World</hello>
      </Asset>
    </XnaContent>
```

Tada! That’s how [IntermediateSerializer](Everything-you-ever-wanted-to-know-about-IntermediateSerializer) represents this particular class in XML.
