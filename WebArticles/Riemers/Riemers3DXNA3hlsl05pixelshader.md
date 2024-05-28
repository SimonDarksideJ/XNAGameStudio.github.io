# Drawing the triangle using custom Shaders

Up to this point, we have a vertex buffer, filled with only 3 vertices defining a single triangle. We also have the metadata: the VertexDeclaration, which describes what kind of data is contained in the vertex data, together with the offset to that kind of data.

We also have a very simple vertex shader. From our vertex stream, it extracts only the position data. For each vertex, this 3D position is transformed to 2D screen coordinates, and passed on to the pixel shader. To perform this transformation, we multiply each vertex with the matrix which is the combination of the View and Projection matrix, which is at this point not yet being specified by our XNA code.

In our XNA app, it is time to load our own effect file we created, and set the transformation matrix. So load the effect file into the Content entry of the Solution Explorer, like you have done before with images. You should see the OurHLSLfile.fx as an additional asset in your Solution Explorer. Find the line in the LoadEffect method that loads my .fx file into the effect variable, and replace it with this line:

```csharp
effect = Content.Load<Effect> ("OurHLSLfile");
```

We’re ready to move on to the Draw method. To draw our triangle using our own technique, we first have to specify our technique and set its parameters. In our case, the only parameter we have to set is xViewProjection, which is the combination of the viewMatrix and the projectionMatrix. So replace the existing code before the line where your Begin your effect with this code:

```csharp
 effect.CurrentTechnique = effect.Techniques["Simplest"];
 effect.Parameters["xViewProjection"].SetValue(viewMatrix*projectionMatrix);
```

Make sure you remove the lines where you try to set other parameters such as xView, xWorld,.. because these don’t exist (yet) in our .fx file.

Our XNA code is ready! However, when you try to run your program, you’ll get an error stating ‘Both a valid vertex shader and pixel shader (or valid effect) must be set on the device before draw operations may be performed‘. This is because although our technique contains a vertex shader, it doesn’t yet contain a valid pixel shader! So let’s go back to our .fx file.

The pixel shader is called for each pixel of the screen that needs to be drawn. In most cases, the pixel shader only needs to calculate the correct color.

The pixel shader receives its input (position and color, in our case) from our vertex shader, and needs to output only color. So let’s define its output structure at the top of our .fx file:

```csharp
struct PixelToFrame
{
    float4 Color        : COLOR0;
};
```

Our first pixel shader will be a very simple method, here it is:

```csharp
PixelToFrame OurFirstPixelShader(VertexToPixel PSIn)
{
    PixelToFrame Output = (PixelToFrame)0;

    Output.Color = PSIn.Color;

    return Output;
}
```

First, an output structure is created, and the color received from the vertex shader is put in the output structure. That’s all it does!

Now we still need to set this method as pixel shader for our technique, at the bottom of the file:

```csharp
PixelShader = compile ps_2_0 OurFirstPixelShader();
```

That’s it! We have 3 colored 3D vertices, pass them to the vertex shader which transforms them into 2D positions and pass them on to the pixel shader, which simply puts the color on the screen. Try to run your XNA program!

You should see the same as the image below: a white triangle. White, because we coded our vertex shader to draw every vertex white. Not a lot of fun, so let’s add some color to it. Go to the vertex shader in your .fx file, and update it to this code:

```csharp
VertexToPixel SimplestVertexShader( float4 inPos : POSITION, float4 inColor : COLOR0)
{
    VertexToPixel Output = (VertexToPixel)0;

    Output.Position = mul(inPos, xViewProjection);
    Output.Color = inColor;

    return Output;
}
```

Note the changes in the first line: now, our vertex shader also expects the vertices to carry color information. In our case, this is OK as the MyOwnVertexFormat contains both position and color information.

The only change that has been made to the interior of the method, is that we simply route this color we get from our XNA app immediately to the output of the vertex shader, instead of simple white.

Now, when you run this code, you should see the same colored triangle as before. Our vertex shader simply transforms the 3D coordinates to 2D screen coordinates, and passes these coordinates together with the correct color to the pixel shader. This pixel shader doesn’t change anything to the color, and passes it on to the screen.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-05Summary1.jpg?raw=true)

I know, I may have rushed this chapter a bit by introducing the pixel shader. This simply is because you need one to show something on the screen. Don’t worry, next chapter we’ll explore the pixel shader in some more depth.

> You can try these exercises to practice what you've learned:
>
> - Adjust your vertex shader so your triangle is rendered in solid yellow.
> - Make the colors in the vertices dependant on their 3D position.

## Code so far

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
         public Color color;
 
         public MyOwnVertexFormat(Vector3 position, Color color)
         {
             this.position = position;
             this.color = color;
         }
 
         public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
              (
                  new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
                  new VertexElement(sizeof(float) * 3, VertexElementFormat.Color, VertexElementUsage.Color, 0)
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
 

            effect = Content.Load<Effect> ("OurHLSLfile");
             SetUpVertices();
             SetUpCamera();
         }
 
         private void SetUpVertices()
         {
             MyOwnVertexFormat[] vertices = new MyOwnVertexFormat[3];
 
             vertices[0] = new MyOwnVertexFormat(new Vector3(-2, 2, 0), Color.Red);
             vertices[1] = new MyOwnVertexFormat(new Vector3(2, -2, -2), Color.Green);
             vertices[2] = new MyOwnVertexFormat(new Vector3(0, 0, 2), Color.Yellow);
 
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

## The HLSL code:

```csharp
float4x4 xViewProjection;

struct VertexToPixel
{
    float4 Position     : POSITION;
    float4 Color        : COLOR0;
};


struct PixelToFrame
{
    float4 Color        : COLOR0;
};

 
 VertexToPixel SimplestVertexShader( float4 inPos : POSITION, float4 inColor : COLOR0)
 {
     VertexToPixel Output = (VertexToPixel)0;
     
     Output.Position = mul(inPos, xViewProjection);
     Output.Color = inColor;
 
     return Output;
 }
 
 
 PixelToFrame OurFirstPixelShader(VertexToPixel PSIn)
 {
     PixelToFrame Output = (PixelToFrame)0;    
 
     Output.Color = PSIn.Color;    
 
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

## Next Steps

[Experimenting with shaders](Riemers3DXNA3hlsl06perpixelcolors)
