# IntermediateSerializer vs. XmlSerializer

Some time before we formally started work on Game Studio 1.0, I was trying to figure out how the Content Pipeline could save model data in XML cache files. My first thought was to use the standard .NET XmlSerializer, but the more I looked into this, the more problems I encountered.
The main problem was size. Model and texture data tends to be pretty big. It often contains huge lists of numbers (such as triangle indices) and vectors (vertex positions or texture colors). To make a usable build system, we had to be able to cache this data in a small and efficient format.

This is what XmlSerializer does if you give it a list containing two vectors:

```xml
  <Vertices>
    <Vertex>
      <X>1</X>
      <Y>2</Y>
      <Z>3</Z>
    </Vertex>
    <Vertex>
      <X>23</X>
      <Y>42</Y>
      <Z>-1</Z>
    </Vertex>
  </Vertices>
  ```

Yikes! I wanted to store that data in a more compact format, perhaps something like:

```xml
  <Vertices>1 2 3 23 42 -1</Vertices>
```

So I started looking at the serializer extensibility model. *XmlSerializer* is pretty flexible: you can use attributes to adjust things, or implement the *IXmlSerializable* interface to control exactly how the XML is written.
Trouble is, the attributes were not powerful enough to do what I wanted. It didn’t seem like a good idea to have our math types implement *IXmlSerializable*, because every game uses vectors, while this XML stuff was only relevant inside the Content Pipeline. If *Vector3* implemented *IXmlSerializable*, that would force every XNA Framework game to load the (rather large) System.Xml assembly. Zune was only a vague possibility back then, but it was a safe guess we might someday want to run on some kind of portable device, and we didn’t want to build such a fundamental memory inefficiency into our API design.
Even if we did make *Vector3* implement *IXmlSerializable*, that wouldn’t help with collections of vectors. There is obviously no way for us to add a custom interface to the standard *List<T>* class, so instead we would have to implement IXmlSerializable on every parent type that might contain such a list. Ick.
Another problem with XmlSerializer was the limited support for polymorphic types. Given a class such as:

```csharp
    class Model
    {
        public object Tag { get; set; }
    }
```

XmlSerializer is unable to serialize the Tag value, because it does not know ahead of time what the data type might be. To make this work, the serializer must be told all the possible types this property could ever contain:

```csharp
    class Model
    {
        [XmlElement(typeof(string))]
        [XmlElement(typeof(bool))]
        [XmlElement(typeof(int))]
        public object Tag { get; set; }
    }
```

Not very scalable! The Content Pipeline object model includes a lot of polymorphic information, such as Tag properties and OpaqueData dictionaries. What if you wanted to set your Tag to a custom, user defined type?

Also, *XmlSerializer* doesn’t support dictionary types, and it can only serialize public fields and properties. Not fatal, but this was irritating for some of the data I was trying to save.
The more time I spent working with *XmlSerializer*, the more problems I ran into. What to do? I made a list of options:

1. Use *XmlSerializer*, and put up with the limitations of an awkward object model, huge cache files, and slow content build
2. Change our design to remove the need for cache files, reducing the quality of incremental rebuilds
3. Use a simpler binary format for the cache files, which would enable fast rebuilds, but would not be human readable and thus useless for debugging
4. Write our own serializer that could work exactly the way we wanted

That last choice was appealing, but also deeply scary. Surely writing a whole new serializer from scratch would be a huge amount of work?
When in doubt, prototype.
I came in to the office one weekend, and spent two days attempting to quantify exactly how hard this would be. By Sunday night I had my answer: easy peasy! In just 48 hours I was able to make a serializer that efficiently saved my data into a compact XML format, exactly the way I wanted it. Of course it took a couple more weeks to handle all the corner cases (generics, nullables, robust error handling, etc) and write unit tests, but it is much easier to get approval for a seemingly risky feature design if you can show your boss an already working prototype implementation.
So that is why *IntermediateSerializer* exists.
I created it to do exactly what I needed for caching intermediate Content Pipeline data. You might find it useful if you have similar requirements:

* Compact XML representation for lists of numeric and vector types
* Handles arbitrary polymorphic data without requiring subtypes to be pre-declared
* Handles shared resources, for serializing cyclic data structures
* Handles external references, automatically fixing up relative filename links from one file to another
* Supports dictionaries
* XML representation can be customized without directly modifying the target types (by implementing a ContentTypeSerializer)

But there are also situations where XmlSerializer would be a better choice:

* Works on Xbox, so you can use it at runtime as well as during your content build
* More flexible attributes for controlling whether data appears as an XML element or attribute
* Supports XML schemas
* Supports SOAP

Next up: more details on how the [IntermediateSerializer](Everything-you-ever-wanted-to-know-about-IntermediateSerializer) works, and how to control it.
