# Everything you ever wanted to know about IntermediateSerializer

It is good to know [how to fish](Teaching-a-man-to-fish), but sometimes you just want to go to the store and buy a salmon that somebody else already caught for you. With this in mind, here is my attempt to fill in the blanks about exactly how to use [IntermediateSerializer](IntermediateSerializervsXmlSerializer).

## Contents

* [The Basics](#the-basics)
* [Inheritance](#inheritance)
* [Including Private Members](#including-private-members)
* [Excluding Public Members](#excluding-public-members)
* [Renaming XML Elements](#renaming-xml-elements)
* [Null References](#null-references)
* [Optional Elements](#optional-elements)
* [AllowNull](#allownull)
* [Collections](#collections)
* [CollectionItemName](#collectionitemname)
* [Dictionaries](#dictionaries)
* [Math Types](#math-types)
* [Polymorphic Types](#polymorphic-types)
* [Namespaces](#namespaces)
* [FlattenContent](#flattencontent)
* [Shared Resources](#shared-resources)
* [External References](#external-references)

## The Basics

Consider this test class:

```csharp
    class TestClass
    {
        public int PublicField = 1;
        protected int ProtectedField = 2;
        private int PrivateField = 3;
        internal int InternalField = 4;

        public string GetSetProperty
        {
            get { return "Hello World"; }
            set { }
        }

        public string GetOnlyProperty
        {
            get { return "Hello World"; }
        }

        public string SetOnlyProperty
        {
            set { }
        }

        public NestedClass Nested = new NestedClass();
    }

    class NestedClass
    {
        public string Name = "Shawn";
        public bool IsEnglish = true;
    }
```

Using the technique described in my [previous post](Teaching-a-man-to-fish), it will create this XML:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <GetSetProperty>Hello World</GetSetProperty>
        <PublicField>1</PublicField>
        <Nested>
          <Name>Shawn</Name>
          <IsEnglish>true</IsEnglish>
        </Nested>
      </Asset>
    </XnaContent>
```

We can draw many conclusions from this:

* The XML always starts with an ```<XnaContent>``` element
* This contains an ```<Asset>``` element, with a Type attribute that specifies the type of the data
* Within the ```<Asset>``` element, all public fields and properties are serialized, using a  separate XML element for each
* Protected, private, or internal data is skipped
* Get-only or set-only properties are skipped
* Properties come before fields
* If there is more than one field or property, these are serialized in the order they were declared
* Nested types are serialized using nested XML elements

## Inheritance

What if we have a class hierarchy where one type derives from another?

```csharp
    class BaseClass
    {
        public int elf = 23;
    }

    class TestClass : BaseClass
    {
        public string hello = "world";
    }
```

This produces:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <elf>23</elf>
        <hello>world</hello>
      </Asset>
    </XnaContent>
```

### Conclusion

* Base class members are serialized before data from the derived type

## Including Private Members

By default, IntermediateSerializer only includes public fields and properties. We can customize this behavior using the [ContentSerializerAttribute](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.content.contentserializerattribute.aspx), which forces even private members to be serialized:

 ```csharp
    class TestClass
    {
        [ContentSerializer]
        private int elf = 23;  // will be serialized
    }
```

## Excluding Public Members

If there are public fields or properties that we do not wish to serialize, we can use the [ContentSerializerIgnoreAttribute](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.content.contentserializerignoreattribute.aspx) to skip them:

```csharp
    class TestClass
    {
        [ContentSerializerIgnore]
        public int elf = 23;  // will not be serialized
    }
```

## Renaming XML Elements

By default, XML elements are named after the field or property which defined them. If you want a different element name, you can specify this using attributes:

```csharp
    class TestClass
    {
        [ContentSerializer(ElementName = "ShawnSaysHello")]
        string hello = "world";

        [ContentSerializer(ElementName = "ElvesAreCool")]
        int elf = 23;
    }
```

Resulting XML:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <ShawnSaysHello>world</ShawnSaysHello>
        <ElvesAreCool>23</ElvesAreCool>
      </Asset>
    </XnaContent>
```

## Null References

If your data contains a null value, for instance:

```csharp
    class TestClass
    {
        public string hello = null;
    }
```

This will produce:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <hello Null="true" />
      </Asset>
    </XnaContent>
```

Note the use of a special XML attribute to indicate the null value. This is different to just not putting any data inside the XML element! The distinction is important for types such as strings, because a null string is not the same thing as a string which is present but empty.

## Optional Elements

You can use attributes to specify that null values should be skipped entirely, rather than serialized with a Null attribute tag. For instance:

```csharp
    class TestClass
    {
        [ContentSerializer(Optional = true)]
        public string a = null;

        public string b = null;

        [ContentSerializer(Optional = true)]
        public string c = string.Empty;
    }
```

Resulting XML:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <b Null="true" />
        <c></c>
      </Asset>
    </XnaContent>
```

### Notes

* Field a has been skipped, because it was null and had the optional attribute
* Field b was not skipped, even though it is null, because it did not have this attribute
* Field c was not skipped, even though it had the attribute, because it was not null

## AllowNull

If you have a field that should never be null, you can specify this with an attribute:

```csharp
    class TestClass
    {
        [ContentSerializer(AllowNull = false)]
        string a;
    }
```

This has no effect when serializing to XML, but it will throw an exception while deserializing if the XML tries to specify a null value.

## Collections

What about collections?

```csharp
    class TestClass
    {
        public string[] StringArray = { "Hello", "World" };
        public List<string> StringList = new List<string> { "This", "is", "a", "test" };
        public int[] IntArray = { 1, 2, 3, 23, 42 };
    }
```

Resulting XML:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <StringArray>
          <Item>Hello</Item>
          <Item>World</Item>
        </StringArray>
        <StringList>
          <Item>This</Item>
          <Item>is</Item>
          <Item>a</Item>
          <Item>test</Item>
        </StringList>
        <IntArray>1 2 3 23 42</IntArray>
      </Asset>
    </XnaContent>
```

Note how IntArray is serialized in a different way to StringArray or StringList. This is a special optimization that IntermediateSerializer applies to collections of numeric types (ints, longs, floats, doubles, bytes, etc). To save space, it uses whitespace to separate each item, rather than XML markup. This is an important part of efficiently handling 3D model data such as vertex and index buffers.

## CollectionItemName

By default, each item in a collection is serialized as an ```<Item>``` element. You can change this using attributes:

```csharp
    class TestClass
    {
        [ContentSerializer(ElementName = "w00t", CollectionItemName = "Flibble")]
        string[] StringArray = { "Hello", "World" };
    }
```

Result:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <w00t>
          <Flibble>Hello</Flibble>
          <Flibble>World</Flibble>
        </w00t>
      </Asset>
    </XnaContent>
```

## Dictionaries

XmlSerializer doesn’t support dictionaries, but IntermediateSerializer does!

 ```csharp
    class TestClass
    {
        public Dictionary<int, bool> TestDictionary = new Dictionary<int, bool>();

        public TestClass()
        {
            TestDictionary.Add(23, true);
            TestDictionary.Add(42, false);
        }
    }
```

Result:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <TestDictionary>
          <Item>
            <Key>23</Key>
            <Value>true</Value>
          </Item>
          <Item>
            <Key>42</Key>
            <Value>false</Value>
          </Item>
        </TestDictionary>
      </Asset>
    </XnaContent>
```

## Math Types

People are often surprised by how IntermediateSerializer handles the XNA Framework math types. Let’s look at some examples:

 ```csharp
     class TestClass
    {
        public Vector3 Vector = new Vector3(1, 2, 3);
        public Rectangle Rectangle = new Rectangle(1, 2, 3, 4);
        public Quaternion Quaternion = new Quaternion(1, 2, 3, 4);
        public Color Color = Color.CornflowerBlue;
        public Vector2[] VectorArray = new Vector2[] { Vector2.Zero, Vector2.One };
    }
```

This will produce:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <Vector>1 2 3</Vector>
        <Rectangle>1 2 3 4</Rectangle>
        <Quaternion>1 2 3 4</Quaternion>
        <Color>FF6495ED</Color>
        <VectorArray>0 0 1 1</VectorArray>
      </Asset>
    </XnaContent>
```

This is similar to the optimization we previously saw being applied to collections of numeric values, using whitespace instead of XML markup to make the encoding more efficient. When you have a collection of math types, such as the vector array shown here, both optimizations are combined.

Colors are represented in hex format, similar to HTML.

## Polymorphic Types

One advantage of IntermediateSerializer over XmlSerializer is that it supports arbitrary polymorphic data without requiring all the possible subtypes to be pre-declared. Let’s look at an example:

 ```csharp
    class A
    {
        public bool Value = true;
    }

    class B : A { }
    class C : A { }

    class TestClass
    {
        public object Hello = "World";
        public object Elf = 23;
        public A[] PolymorphicArray = { new A(), new B(), new C() };
    }
```

Resulting XML:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <Hello Type="string">World</Hello>
        <Elf Type="int">23</Elf>
        <PolymorphicArray>
          <Item>
            <Value>true</Value>
          </Item>
          <Item Type="B">
            <Value>true</Value>
          </Item>
          <Item Type="C">
            <Value>true</Value>
          </Item>
        </PolymorphicArray>
      </Asset>
    </XnaContent>
```

XML Type attributes are used to indicate the types of polymorphic data. The Hello and Elf fields need this attribute, because they are declared as type object, but then contain data of a more specialized subtype.

Note how in the PolymorphicArray, the first item (of type A) is not polymorphic, but the second two (of types B and C) are. This is because I declared it as an array of A, so only the items that are subclasses of A require a Type attribute. If I had declared it as an array of object, all three items would require this attribute.

## Namespaces

If your XML uses a lot of Type attributes, it can get pretty boring having to enter long names like “Microsoft.Xna.Framework.Graphics.Color” over and over again. IntermediateSerializer can cut down this redundancy by using XML namespaces as shortcuts for C# namespaces. You can think of this as an XML equivalent of the C# “using” keyword.

For instance this class:

 ```csharp
     namespace This.Is.A.Long.Namespace.For.A.Test
    {
        class TestHelper
        {
            public bool Value = true;
        }

        class TestClass
        {
            public object A = new TestHelper();
            public object B = Vector2.Zero;
            public object C = Color.CornflowerBlue;
        }
    }
```

Can be serialized like so:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent xmlns:Test="This.Is.A.Long.Namespace.For.A.Test"
                xmlns:Framework="Microsoft.Xna.Framework"
                xmlns:Graphics="Microsoft.Xna.Framework.Graphics">
      <Asset Type="Test:TestClass">
        <A Type="Test:TestHelper">
          <Value>true</Value>
        </A>
        <B Type="Framework:Vector2">0 0</B>
        <C Type="Graphics:Color">FF6495ED</C>
      </Asset>
    </XnaContent>
```

The three xmlns attributes define namespace shortcuts, after which we can refer to types using the abbreviated syntax “Test:TestHelper”, “Framework:Vector2”, etc.

This is optional: you could expand out the namespaces and the XML would load exactly the same. When you save an XML file, IntermediateSerializer uses some heuristics to choose namespace shortcuts, as shown in the above example. If you are writing the XML by hand, you are free to define any shortcuts you like.

## FlattenContent

By default, every nested class or collection will open up a nested XML element. You can change this behavior using attributes. For instance this class:

 ```csharp
     class TestClass
    {
        [ContentSerializer(FlattenContent = true)]
        public NestedClass Nested = new NestedClass();

        [ContentSerializer(FlattenContent = true, CollectionItemName = "Boo")]
        public string[] Collection = { "Hello", "World" };
    }

    class NestedClass
    {
        public string Name = "Shawn";
        public bool IsEnglish = true;
    }
```

Produces this XML:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <Name>Shawn</Name>
        <IsEnglish>true</IsEnglish>
        <Boo>Hello</Boo>
        <Boo>World</Boo>
      </Asset>
    </XnaContent>
```

If I left out the FlattenContent attributes, it would produce this instead:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <Nested>
          <Name>Shawn</Name>
          <IsEnglish>true</IsEnglish>
        </Nested>
        <Collection>
          <Item>Hello</Item>
          <Item>World</Item>
        </Collection>
      </Asset>
    </XnaContent>
```

## Shared Resources

The default behavior of representing nested classes as nested XML elements works fine as long as you limit yourself to tree-like data, but is not capable of representing circular data structures. For instance if Foo contained a reference to Bar, and then Bar contained a reference back to Foo, the serializer would nest Bar inside Foo, and then another copy of Foo inside Bar, with another Bar inside that… Infinite recursion = stack overflow = badness.

Solution: use attributes to specify that your circular references should be serialized as shared resources. Consider this circular linked list:

 ```csharp
     class TestClass
    {
        [ContentSerializer(SharedResource = true)]
        public Linked Head;

        public TestClass()
        {
            Head = new Linked();
            Head.Value = 1;

            Head.Next = new Linked();
            Head.Next.Value = 2;

            Head.Next.Next = new Linked();
            Head.Next.Next.Value = 3;

            Head.Next.Next.Next = Head;
        }
    }

    class Linked
    {
        public int Value;

        [ContentSerializer(SharedResource = true)]
        public Linked Next;
    }
```

If I left out the SharedResource attributes, serializing this data would throw an exception complaining that it is a cyclic data structure. But with the attributes, we get this XML:

 ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <XnaContent>
      <Asset Type="TestClass">
        <Head>#Resource1</Head>
      </Asset>
      <Resources>
        <Resource ID="#Resource1" Type="Linked">
          <Value>1</Value>
          <Next>#Resource2</Next>
        </Resource>
        <Resource ID="#Resource2" Type="Linked">
          <Value>2</Value>
          <Next>#Resource3</Next>
        </Resource>
        <Resource ID="#Resource3" Type="Linked">
          <Value>3</Value>
          <Next>#Resource1</Next>
        </Resource>
      </Resources>
    </XnaContent>
```

See what’s going on here? Rather than being embedded directly at the point where they are referenced, the shared resource objects have been gathered up into a ```<Resources>``` element, which comes after the regular ```<Asset>``` data. Resources are identified by unique ID strings. In this case the serializer has generated ID values  using a simple naming convention (#Resource1, #Resource2, etc) but you can use any string you like if you are writing the XML by hand.

## External References

The final thing I want to talk about today is how IntermediateSerializer handles the Content Pipeline ExternalReference type. This test class:

 ```csharp
     class TestClass
    {
        public ExternalReference<Texture2D> Texture = new ExternalReference<Texture2D>("grass.tga");
        public ExternalReference<Effect> Shader = new ExternalReference<Effect>("foliage.fx");
    }
```

Produces this XML:

```xml
<?xml version="1.0" encoding="utf-8"?>
<XnaContent>
  <Asset Type="TestClass">
    <Texture>
      <Reference>#External1</Reference>
    </Texture>
    <Shader>
      <Reference>#External2</Reference>
    </Shader>
  </Asset>
  <ExternalReferences>
    <ExternalReference ID="#External1" TargetType="Microsoft.Xna.Framework.Graphics.Texture2D">grass.tga</ExternalReference>
    <ExternalReference ID="#External2" TargetType="Microsoft.Xna.Framework.Graphics.Effect">foliage.fx</ExternalReference>
  </ExternalReferences>
</XnaContent>
```

Similar to shared resources, the ExternalReference data has been gathered up into a special ```<ExternalReferences>``` element at the end of the file, outside the regular ```<Asset>``` data. Each reference is identified by a unique ID string, which is generated by the serializer when it writes XML, but can be anything you like if you are writing the XML by hand.

There are two reasons for this seemingly crazy encoding:

Firstly, IntermediateSerializer applies special logic to the filenames used by ExternalReference objects. In memory, ExternalReference always stores filenames in absolute format. This keeps things nice and simple, avoiding the confusion that can occur with relative filenames if you lose track of what root directory they are supposed to be relative to. But on disk, filenames are stored relative to the XML file which contains them. This allows you to move an entire content directory, containing many files which reference each other, to a different location, and still have the references resolve correctly. IntermediateSerializer handles the translation between absolute and relative formats during the Serialize and Deserialize operations. This is the purpose of the third parameter to the Serialize method, which tells it what directory references should be relative to. If you don’t specify that, the output XML will contain absolute filenames.

Secondly, gathering the ExternalReference data together at the end of the file makes it easy for other tools to mine this information. Even without knowing anything about what a “TestClass” is, I could write a program that scanned the above XML, skipped over the entire ```<Asset>``` tag, and extracted the target type and filename of the two references. This could be useful for things like dependency analysis or source control management. For instance it would be trivial to write a tool that scanned the obj directory after a build and answered questions like “which models are using texture X?”. We’ve never actually written such a tool, but who knows? It seemed like a good idea to make this data easily available in case we someday needed it.
