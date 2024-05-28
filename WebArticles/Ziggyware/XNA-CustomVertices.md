# Creating a Custom Vertex in XNA


Here is a declaration of a custom vertex that implements position, texture, normal, tangent and bitangent properties.

The declaration also has to implement the "*IVertexType*" interface, which details how the vertex declaration is returned to the effect.

Make the vertex Serializable into XML (or Binary) by using the [Serializable] Attribute:

```csharp
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using System;

    [Serializable]
    public struct VertexPosTexNormalTanBitan : IVertexType
    {
```


Now we declare the member elements of the vertex

```csharp
        Vector3 pos;
        Vector2 tex;
        Vector3 normal, tan, bitan;
```


Create a static member to hold the VertexDeclaration definition with the VertexElements supported by the declaration

```csharp
    public static readonly VertexDeclaration VertexDeclaration = new VertexDeclaration
    (
        new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
        new VertexElement(sizeof(float) * 3, VertexElementFormat.Vector2, VertexElementUsage.TextureCoordinate, 0),
        new VertexElement(sizeof(float) * 5, VertexElementFormat.Vector3, VertexElementUsage.Normal, 0),
        new VertexElement(sizeof(float) * 8, VertexElementFormat.Vector3, VertexElementUsage.Tangent, 0),
        new VertexElement(sizeof(float) * 11, VertexElementFormat.Vector3, VertexElementUsage.Binormal, 0)
    );
```


And a constructor that sets the member variables

```csharp
    public VertexPosTexNormalTanBitan(Vector3 position, 
                                      Vector2 uv, 
                                      Vector3 normal, 
                                      Vector3 tan, 
                                      Vector3 bitan)
    {
        pos = position;
        tex = uv;
        this.normal = normal;
        this.tan = tan;
        this.bitan = bitan;
    }
```

As per the IVertexType Interface, you need to implement the VertexDeclaration property for the Custom Vertex type as follows:

```csharp
    VertexDeclaration IVertexType.VertexDeclaration
    {
        get { return VertexDeclaration; }
    }
```

You can implement the != and == operators (**not required**).

```csharp
    public static bool operator !=(VertexPosTexNormalTanBitan left, 
                                      VertexPosTexNormalTanBitan right)
    {
        return left.GetHashCode() != right.GetHashCode();
    }

    public static bool operator ==(VertexPosTexNormalTanBitan left, 
                                        VertexPosTexNormalTanBitan right)
    {
        return left.GetHashCode() == right.GetHashCode();
    }
```


As well as override the Equals() Method (**not require**).

```csharp
    public override bool Equals(object obj)
    {
        return this == (VertexPosTexNormalTanBitan)obj;
    }
```


Lets declare some accessors for the member variables

```csharp
    public Vector3 Position { get { return pos; } set { pos = value; } }
    public Vector3 Normal { get { return normal; } set { normal = value; } }
    public Vector2 Tex { get { return tex; } set { tex = value; } }
    public Vector3 Tan { get { return tan; } set { tan = value; } }
    public Vector3 Bitan { get { return bitan; } set { bitan = value; } }
```


A custom vertex can also implement the SizeInBytes() method to make it easier to create a VertexDeclaration based on the vertex type.

```csharp
    public static int SizeInBytes { get { return sizeof(float) * 14; } }
```


As well as the GetHashCode() method (not required).

```csharp
    public override int GetHashCode()
    {
        return pos.GetHashCode() | 
                 tex.GetHashCode() | 
                 normal.GetHashCode() | 
                 tan.GetHashCode() | 
                 bitan.GetHashCode();
    }
```


Finally we wrap it up by making a custom ToString() method to display the vertex in the debugger a little nicer (**not required**).

```csharp
        public override string ToString()
        {
            return string.Format("{0},{1},{2}", pos.X, pos.Y, pos.Z);
        }
    }
```


Thats all there is to creating your own custom vertices for XNA :)