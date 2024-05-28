# The first Vertex Shader

Last chapter we’ve seen how to create our own vertex format. This chapter you will write your first HLSL code, together with your first vertex shader.

This would be a nice moment to have another look at our flowchart below. You notice the big arrow from our XNA app toward the vertex shader. At this point, we have the vertex stream (the position and color of our 3 vertices), as well as the metadata, which describes what’s in this vertex stream together with the memory size of each vertex.

When looking at the image, you’ll see it’s time to begin coding on our vertex shader!

![HLSL Overview](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-02Hlsl1.jpg?raw=true)

Although one of the main goals of my tutorials is to keep all code in 1 file, we cannot get around this one. You’ll have to create a new empty effect file and give it a name (I named mine OurHLSLfile.fx). To do this, right-click on the Content entry of your XNA Solution Explorer and select Add -> New Item. In the dialog, select Effect file, give it the OurHLSLfile.fx name and click Add.

You should note the file has been added to the Content entry of your XNA Project. You will be presented with a screen containing a lot of code that seems pretty unknown, so delete it all

Although HLSL is not 100% the same as C# code, you will have absolutely no problem reading and writing the code. I could give you an extremely dry summary of the HLSL grammar, but I prefer to introduce you the syntax by showing some examples. At the end of this Series, you’ll be able to read and write almost any HLSL code you want.

As you have experienced throughout the previous series, an .fx file can describe one or more techniques. One technique can have multiple passes as you’ll see in the next chapters, but let’s start by defining a simple technique with only one pass. You can already put this as your first HLSL code in your .fx file:

```csharp
technique Simplest
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 SimplestVertexShader();
        PixelShader = NULL;
    }
}
```

This defines a technique, Simplest, which has one pass. This pass has a vertex shader (SimplestVertexShader), but no pixelshader. This indicates our vertex shader will pass its output data to the default pixel shader.

The main task of a 3D vertex shader is to accept a vertex with a 3D position, and to processor this into a 2D screen coordinate. Optionally, the vertex shader can also produce extra data, such as the color or texture coordinate.

Before we start coding our vertex shader, we would better define a structure to hold the data our vertex shader will send to the default pixel shader. The vertex shader method we will create, SimplestVertexShader, will simply transform the 3D positions in the vertices it receives from our XNA app to 2D screen pixel coordinates, and send them together with the color to the pixel shader, so put this code at the top of your .fx file:

```csharp
struct VertexToPixel
{
    float4 Position     : POSITION;
    float4 Color        : COLOR0;
};
```

This again looks very much like C#, only for the :POSITION and :COLOR0. These are called semantics and indicate how our GPU should use the data. More on this in the next paragraph. Let’s start our SimplestVertexShader method, so you’ll better understand this. Place this method between the structure definition and our technique definition:

```csharp
VertexToPixel SimplestVertexShader(float4 inPos : POSITION)
{
    VertexToPixel Output = (VertexToPixel)0;

    Output.Position = mul(inPos, xViewProjection);
    Output.Color = 1.0f;

    return Output;
}
```

This again looks a lot like C#. The first line indicates our method (our vertex shader) will generate a filled VertexToPixel structure. It also indicates the vertices that XNA sends to your vertex shader should contain positional data, as indicates by the POSITION semantic. This is very important: it links the data inside your vertex stream (as indicated in your VertexDeclaration) to your HLSL code.

Keep in mind that this method is called for every vertex in your vertex stream. The first line in the method creates an empty output structure. The second line takes the 3D coordinates of the vertex, and transforms them to 2D screen coordinates by multiplying them by the combination of the View and Projection matrix. For more information on this, you can always have a look at the Matrix sessions in my ‘Extra Reading’ section, which you can find at the right of every page.

Then we fill the Color member of the output structure. When you look at the definition of our output structure, you’ll see this has to be a float4: one float for each of the 3 color components, and an extra float for the alpha (transparency) value. You could fill this color by using the following code:

```csharp
Output.Color.r = 1.0f;
Output.Color.g = 0.0f;
Output.Color.b = 1.0f;
Output.Color.a = 1.0f;
```

This would indicate purple, as you combine red and blue. The following code specifies exactly the same:

```csharp
Output.Color.rba = 1.0f;
Output.Color.g = 0.0f;
```

This is called a swizzle, and helps you to code faster. Instead of rgba, you can also use xyzw. The rgba swizzle is usually used when working with colors, while the xyzw swizzle is used in combination with coordinates, but they do exactly the same. You can also use indices, which is useful for use in an algorithm:

```csharp
Output.Color[0] = 1.0f;
Output.Color[1] = 0.0f;
Output.Color[2] = 1.0f;
Output.Color[3] = 1.0f;
```

In our example vertex shader above, we simply set Output.Color = 1.0f, which means the 4 components of the color are all set to 1.0f, corresponding to white. So our vertex shader will transform our 3D vertices to 2D screen coordinates, and pass them together with the color white to the default pixel shader. This means in our case of 1 triangle, our pixel shader will draw a solid white triangle to the window.

There’s still something missing. We still need to define xViewProjection, the matrix we’re using in our vertex shader to transform the 3D coordinate to the 2D screen coordinate. This matrix depends on the View and Projection matrices of the camera, so they stay the same for all vertices rendered in one frame. This matrix need to be specified by your XNA code, so put this at the very top of your HLSL code:

```csharp
float4x4 xViewProjection;
```

This indicates xViewProjection is a matrix with 4 rows and 4 columns, so it can hold a standard XNA matrix. Our XNA app will fill this matrix in the next chapter.

That’s it for our first HLSL code! Of course, we still need to call the technique from our XNA app, as well as set the xViewProjection matrix. Because this chapter would otherwise become too lengthy, we’ll discuss the XNA part in the next chapter.

## Code so far

Here you can find already what you should have as HLSL code:

```csharp
float4x4 xViewProjection;

struct VertexToPixel
{
    float4 Position     : POSITION;
    float4 Color        : COLOR0;
};

VertexToPixel SimplestVertexShader(float4 inPos : POSITION)
{
    VertexToPixel Output = (VertexToPixel)0;

    Output.Position = mul(inPos, xViewProjection);
    Output.Color = 1.0f;
    
    return Output;
}

technique Simplest
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 SimplestVertexShader();
        PixelShader = NULL;
    }
}
```

## Next Steps

[Drawing the triangle using custom Shaders](Riemers3DXNA3hlsl05pixelshader)
