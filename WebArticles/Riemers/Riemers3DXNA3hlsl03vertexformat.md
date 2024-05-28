# Defining our own vertex format

Let’s have a look at the staring point in our flowchart: the big arrow starting in our XNA app, going to our vertex shader:

![HLSL Overview](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-02Hlsl1.jpg?raw=true)

It represents the flow of vertex data from our XNA app to the vertex shader on the graphics card. This is triggered every time we issue some kind of draw command in XNA. When we pass the vertex data from our XNA app to our vertex shader, we need to pass some information with it, describing what kind of data is contained in the vertex stream. Using shaders, you need to specify exactly what information can be found in the stream, and where.

So what we need is:

* A structure that can hold the necessary data for each vertex and
* A definition of the data, so the vertex shader knows which data is included with every vertex.

In the starting code, we’ve been using the VertexPositionColor struct which satisfies both requirements. Here, we’re going to define a new struct, MyOwnVertexFormat, that will be exactly the same as the VertexPositionColor struct, to see what’s in there and why it is needed. This will allow us to expand it further on in this series.

> For a far more detailed explanation on creating custom vertex formats, read Recipe 5-14.

To satisfy the 2 requirements, in the example of a simple colored triangle, we need our vertices to hold 3D Position data as well as Color data. So this is how we start our struct (you can put this at the top of our code):

```csharp
 struct MyOwnVertexFormat
 {
     private Vector3 position;
     private Color color;

     public MyOwnVertexFormat (Vector3 position, Color color)
     {
         this.position = position;
         this.color = color;
     }
 }
```

We simply defined a new structure, and defined it so it can hold a Vector3 and a Color. We also defined a constructor, so later in our program we can create and fill a new instance of this struct in one line. The first requirement has been satisfied.

Next, we need a way to tell our graphics card that the data we are sending actually contain Position and Color data. Although we gave the elements straightforward names (position, color), the graphics card needs to be told explicitly which data it will receive.

As seen in the previous series, this is done by means of setting a VertexDeclaration before rendering triangle. However, when we created this VertexDeclaration, we always used the VertexElements property of the type of vertices we were rendering from. Since we are defining our own kind of vertices, we also need to define its VertexElements.

The VertexDeclaration will contain 1 entry for each type of data accompanying each vertex. Put this code inside our struct, at the very end:

```csharp
 public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
 (
     new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
     new VertexElement(sizeof(float) * 3, VertexElementFormat.Color, VertexElementUsage.Color, 0)
 );
```

For each type of data, we define what it is used for, and where it can be found. The following paragraph contains a brief summary of the explanation in Recipe 5-14.

The first argument is very important. It indicates at which offset IN BYTES the type of data can found in the vertex stream. So the first type of data, the 3D position, starts at offset 0.

Next, we indicate how that kind of data is stored, such as int, float1, short4 and more. A position comes in a Vector3 (which is composed of 3 floats). The next argument is the most important one: it describes the kind of information, such as position, color, texture coordinate, tangent, etc you are passing. This is needed, so XNA can automatically link the right data from our vertex stream to the right variables in our vertex shaders.

For the last argument, suppose you would like to add 2 textures to a triangle. This implicates you would have to pass 2 sets of texture coordinates along with each vector. For this, you can use the last argument, which is the index of each info. Since we’ll only be passing 1 position and 1 color for each vertex, this will be 0 for both lines.

For our color, we do pretty much the same. It is still part of vertexstream 0, but because it is preceded by the position it can’t be found at offset 0. The position consists of 3 floats, this is what we need to indicate (a float occupies 4 bytes, so sizeof(float)*3 is 12, which is the offset in bytes to the color information). Although a color is in fact a Vector4 (4 floats: the RGBA values), we need to specify Color, because each value needs to be mapped within the range [0..1] before it can be passed to the vertex shader as a color. This is an exception.

Now let’s change our code so we’ll be using our MyOwnVertexFormat instead of the VertexPositionColor struct. Change our SetUpVertices to this:

```csharp
 private void SetUpVertices()
 {
     MyOwnVertexFormat[] vertices = new MyOwnVertexFormat[3];

     vertices[0] = new MyOwnVertexFormat(new Vector3(-2, 2, 0), Color.Red);
     vertices[1] = new MyOwnVertexFormat(new Vector3(2, -2, -2), Color.Green);
     vertices[2] = new MyOwnVertexFormat(new Vector3(0, 0, 2), Color.Yellow);

     vertexBuffer = new VertexBuffer(device, MyOwnVertexFormat.VertexDeclaration, vertices.Length, BufferUsage.WriteOnly);
     vertexBuffer.SetData(vertices);
 }
```

Easy enough, since we provided all details needed in our struct. Even better, since the vertexbuffer knows which kind of vertices it stores, we don’t have to change anything else.

OK, we have recreated the pre-built VertexPositionColor struct. When you run the code, you should see the same triangle. Only this time, you know how what the VertexDeclaration contains and, more important, you know how to extend a vertex format. We will practice this in a later chapter.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-03Summary1.jpg?raw=true)

This is what the XNA code should look like, with the integration of our own vertex format:

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
 

            effect = Content.Load<Effect> ("effects"); SetUpVertices();            SetUpCamera();
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
 
             effect.CurrentTechnique = effect.Techniques["ColoredNoShading"];
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
 
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

[The first Vertex Shader](Riemers3DXNA3hlsl04vertexshader)
