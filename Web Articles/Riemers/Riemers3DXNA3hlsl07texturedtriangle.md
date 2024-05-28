# Texturing our triangle using the Pixel Shader

When you take another look at our flowchart, you’ll see we’ve already seen the most part of it. We’ve covered pretty much everything starting from our vertex stream to the output of the pixel shader. We’ve also set a shader constant, xViewProjection, from within our XNA app. This means we have implemented the Colored technique from my default effects.fx file!

A logical next step would be to load a texture from within our XNA app, and have our pixel shader sample the correct color for each pixel.

The first part would be to load the texture in our XNA app, and to update the vertex stream as well as the VertexDeclaration, so they also send texture coordinate information to the vertex shader.

We will immediately start by loading our street texture, which you can download here. I got them from this site, it has a lot of very nice textures you can use in your own app. You can already put this line at the top of your XNA code:

```csharp
 Texture2D streetTexture;
```

Import the image into your Solution Explorer as seen in the chapter Textures of Series 2. Add this line to our LoadContent method:

```csharp
streetTexture = Content.Load<Texture2D> ("streettexture");
```

The next thing to do would be to update the MyOwnVertex structure at the top of our code, so it can handle texture coordinates. We’ll remove the Color entry, as we’ll no longer use it:

```csharp
 struct MyOwnVertexFormat
 {
     private Vector3 position;
     private Vector2 texCoord;

     public MyOwnVertexFormat(Vector3 position, Vector2 texCoord)
     {
         this.position = position;
         this.texCoord = texCoord;
     }
 }
```

To specify the position in a texture, you need a X and Y coordinate, so we’ll store a Vector2.

Now each vertex can now hold a position as well as a texture coordinate, so let’s update them in our SetUpVertices method:

```csharp
 vertices[0] = new MyOwnVertexFormat(new Vector3(-2, 2, 0), new Vector2(0.0f, 0.0f));
 vertices[1] = new MyOwnVertexFormat(new Vector3(2, -2, -2), new Vector2(0.125f, 1.0f));
 vertices[2] = new MyOwnVertexFormat(new Vector3(0, 0, 2), new Vector2(0.25f, 0.0f));
```

This defines the 3D position as well as the 2D texture coordinate of our 3 vertices. Remember, to have this correctly connected to your vertex shader, you also need to update your VertexElements accordingly:

```csharp
 public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
  (
     new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
     new VertexElement(sizeof(float) * 3, VertexElementFormat.Vector2, VertexElementUsage.TextureCoordinate, 0)
 );
```

You see we have replaced the Color entry by this TextureCoordinate entry, which is stored in a Vector2. The second argument has remained the same, as the texture coordinate can still be found at the same position as where the color was: immediately after the positional data. As second argument, we also specify that XNA should reserve 2 floats of memory to store the coordinates.

So far for the XNA part, let’s turn to our HLSL file again. Before moving on to our shaders, let’s first add these lines to the top of our HSLL code:

```csharp
Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
```

The first line defines a variable that will hold our texture. We’ll need to fill this variable from within our XNA app. The second line sets up the sampler. A sampler is linked to a texture, and describes how the texture should be processed. We set the min- and magfilters, together with the mipfilter, to linear, so we’ll always get nicely shader colors, even when the camera is very close to the triangle.

> See Recipe 5-2 for examples on all different kinds of texture addressing modes, and the note in Recipe 3-7 for the what and why on mipmaps.

We set the texture coordinate states to mirror, which means that, for example, texture coordinate (2.2f, 1.4f) will be automatically mapped to the [0,1] region and will thus be replaced by (0.2f, 0.6f).

Next, we’ll instruct our vertex shader to simply route the texture coordinates from its input to its output. Therefore, we first need to adjust its output structure, VertexToPixel, so our vertex shader is expected to generate a texture coordinate instead of a color:

```csharp
struct VertexToPixel
{
    float4 Position     : POSITION;
    float2 TexCoords    : TEXCOORD0;
};
```

Once again, we’re using the TEXCOORD0 semantic to pass additional data from our vertex shader to our pixel shader. Although we’re only passing 2 floats instead of the maximum 4, this is the correct choice. Now update our vertex shader to this:

```csharp
VertexToPixel SimplestVertexShader( float4 inPos : POSITION, float2 inTexCoords : TEXCOORD0)
{
    VertexToPixel Output = (VertexToPixel)0;

    Output.Position =mul(inPos, xViewProjection);
    Output.TexCoords = inTexCoords;

    return Output;
}
```

2 major changes:

- The first line indicates that the shader expects the vertices to carry TEXCOORD0 information
- This information is immediately routed towards the output of the vertex shader, the interpolator.

Think of what the pixel shader receives from the interpolater: the interpolated 2D screen position and the interpolated 2D texture coordinate. The pixel shader needs to output the pixel color, which will be sampled from our texture at the correct position.

So change the line in your pixel shader to this:

```csharp
Output.Color = tex2D(TextureSampler, PSIn.TexCoords);
```

This command simply retrieves the color of the pixel in the xTexture image, corresponding to the 2D coordinate in PSIn.TexCoords.

There remains only one thing to do: set the xTexture from within our XNA app. So add this line to our Draw method:

```csharp
 effect.Parameters["xTexture"].SetValue(streetTexture);
```

Which loads the streetTexture variable of our XNA app into the xTexture XNA-to-HLSL variable of our HLSL code.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-07Summary1.jpg?raw=true)

By now, you should have an idea of how the XNA app, the vertex shader and the pixel shader interact with each other. Next chapter, I’ll discuss something XNA specific again, because we won’t succeed in achieving the final image of this Series without expanding our scene. After that one, we’ll go back to HLSL..

You can try these exercises to practice what you've learned:
In your vertex, override the texture coordinates so the xy coordinates of the 3D postion are stored as xy texture coordinates. This should add a lot of copies of the texture, see the next chapters to learn why.

## The HLSL code

```csharp
float4x4 xViewProjection;


 Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
struct VertexToPixel
{
    float4 Position     : POSITION;    
    float2 TexCoords    : TEXCOORD0;
};


struct PixelToFrame
{
    float4 Color        : COLOR0;
};


 VertexToPixel SimplestVertexShader( float4 inPos : POSITION, float2 inTexCoords : TEXCOORD0)
 {
     VertexToPixel Output = (VertexToPixel)0;
     
     Output.Position =mul(inPos, xViewProjection);
     Output.TexCoords = inTexCoords;
 
     return Output;
 }
 
 PixelToFrame OurFirstPixelShader(VertexToPixel PSIn)
 {
     PixelToFrame Output = (PixelToFrame)0;
 
     Output.Color = tex2D(TextureSampler, PSIn.TexCoords);


    return Output;
}

technique Simplest
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 SimplestVertexShader();
        PixelShader = compile ps_2_0 OurFirstPixelShader();
    }
}
```

## The XNA code

```csharp
 using System;
 using System.Collections.Generic;
 using Microsoft.Xna.Framework;
 using Microsoft.Xna.Framework.Audio;
 using Microsoft.Xna.Framework.Content;
 using Microsoft.Xna.Framework.GamerServices;
 using Microsoft.Xna.Framework.Graphics;
 using Microsoft.Xna.Framework.Input;
 using Microsoft.Xna.Framework.Net;
 using Microsoft.Xna.Framework.Storage;
 
 namespace XNAseries3
 {
     public struct MyOwnVertexFormat
     {
         public Vector3 position;
         private Vector2 texCoord;
 
         public MyOwnVertexFormat(Vector3 position, Vector2 texCoord)
         {
             this.position = position;
             this.texCoord = texCoord;
         }
 
         public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
              (
                  new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
                  new VertexElement(sizeof(float) * 3, VertexElementFormat.Vector2, VertexElementUsage.TextureCoordinate, 0)
              );
     }
 
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         GraphicsDeviceManager graphics;
         GraphicsDevice device;
 
         Effect effect;
         Matrix viewMatrix;
         Matrix projectionMatrix;
         VertexBuffer vertexBuffer;
         Vector3 cameraPos;
         Texture2D streetTexture;
 
         public Game1()
         {
             graphics = new GraphicsDeviceManager(this);
             Content.RootDirectory = "Content";
         }
 
         protected override void Initialize()
         {
             graphics.PreferredBackBufferWidth = 500;
             graphics.PreferredBackBufferHeight = 500;
             graphics.IsFullScreen = false;
             graphics.ApplyChanges();
             Window.Title = "Riemer's XNA Tutorials -- Series 3";
 
             base.Initialize();
         }
 
         protected override void LoadContent()
         {
             device = GraphicsDevice;
 

            effect = Content.Load<Effect> ("OurHLSLfile");            SetUpVertices();
            SetUpCamera();



            streetTexture = Content.Load<Texture2D> ("streettexture");
         }
 
         private void SetUpVertices()
         {
             MyOwnVertexFormat[] vertices = new MyOwnVertexFormat[3];
 
             vertices[0] = new MyOwnVertexFormat(new Vector3(-2, 2, 0), new Vector2(0.0f, 0.0f));
             vertices[1] = new MyOwnVertexFormat(new Vector3(2, -2, -2), new Vector2(0.125f, 1.0f));
             vertices[2] = new MyOwnVertexFormat(new Vector3(0, 0, 2), new Vector2(0.25f, 0.0f));
 
             vertexBuffer = new VertexBuffer(device, MyOwnVertexFormat.VertexDeclaration, vertices.Length, BufferUsage.WriteOnly);
             vertexBuffer.SetData(vertices);
         }
 
         private void SetUpCamera()
         {
             cameraPos = new Vector3(0, 5, 6);
             viewMatrix = Matrix.CreateLookAt(cameraPos, new Vector3(0, 0, 1), new Vector3(0, 1, 0));
             projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 200.0f);
         }
 
         protected override void UnloadContent()
         {
         }
 
         protected override void Update(GameTime gameTime)
         {
             if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                 this.Exit();
 
             base.Update(gameTime);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);
 
             effect.CurrentTechnique = effect.Techniques["Simplest"];
             effect.Parameters["xViewProjection"].SetValue(viewMatrix * projectionMatrix);
             effect.Parameters["xTexture"].SetValue(streetTexture);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.SetVertexBuffer(vertexBuffer);
                 device.DrawPrimitives(PrimitiveType.TriangleList, 0, 1);
             }
 
             base.Draw(gameTime);
         }
     }
 }
```

## Next Steps

[Triangle Strips](Riemers3DXNA3hlsl08trianglestrip)
