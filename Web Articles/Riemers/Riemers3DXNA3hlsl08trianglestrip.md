# Higher performance by using Triangle Strips -– Texture Mirroring revisited

This chapter has nothing to do with HLSL – I’ve written it so people who are not following the Series can understand how to use triangle strips just by reading this chapter.

Up till now, we’ve only drawn a single triangle. This chapter we’ll expand our scene, so it’ll look a bit more like a street. I’ve divided the scene into 5 parts: the road, the 2 sides of the pavement border, the pavement itself and the wall. 5 textured quads, which have to be drawn by 10 textured triangles.

To draw these 10 triangles, we could simply define 30 vertices, each holding their 3D position and 2D texture coordinate, which can be presented this way:

![Triangle Strip](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-08TriangleStrip1.jpg?raw=true)

I’ve only put a few vertex numbers on the picture; otherwise it would be a mess. Every triangle has 3 vertices, and the vertices are all declared in a clockwise manner relative to the camera.

By looking at the image, I hope you notice almost all vertices are declared 2 or 3 times! This means a lot of redundancy in the information we send over to our PCI express slot, so there must be a way we can reduce the amount of vertices we send. For cases like this, where every triangle is sharing 2 vertices with the previous one, the TriangleStrip should be used. The idea is illustrated in the image below:

![Triangle Strip](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-08TriangleStrip2.jpg?raw=true)

The idea behind TriangleStrips is that you should define each vertex only once. So for the first triangle, you have to define vertices 0,1 and 2. Now, for the next triangle, you only have to add vertex 3! XNA will always use the last 3 vertices to draw the triangle, so for the second triangle it uses vertices 1, 2 and 3, which is correct.

For the fourth triangle, you only have to add vertex 4, and XNA will use vertices 2,3 and 4. In a formula, the n-th triangle is defined by the (n-1)-th, the n-th and the (n+1)-th vertex, with n starting from 1. This way, you see the total amount of vertices has been decreased by a huge amount! Of course, this can yield much higher framerates when working with a larger number of triangles, such as our terrain of Series 1 (which could indeed also be defined using a TriangleStrip, see Recipe 5-8).

There’s still one problem remaining. When you look at the red arrows, you’ll see it’s impossible to define all vertices in a clockwise manner around your triangles. This will always be the case, so when using a TriangleStrip, the rule is you switch your vertex definition from clockwise to counterclockwise and back every triangle.

Let’s change the vertex definitions:

```csharp
 MyOwnVertexFormat[] vertices = new MyOwnVertexFormat[12];

 vertices[0] = new MyOwnVertexFormat(new Vector3(-20, 0, 10), new Vector2(-0.25f, 25.0f));
 vertices[1] = new MyOwnVertexFormat(new Vector3(-20, 0, -100), new Vector2(-0.25f, 0.0f));
 vertices[2] = new MyOwnVertexFormat(new Vector3(2, 0, 10), new Vector2(0.25f, 25.0f));
 vertices[3] = new MyOwnVertexFormat(new Vector3(2, 0, -100), new Vector2(0.25f, 0.0f));
 vertices[4] = new MyOwnVertexFormat(new Vector3(2, 1, 10), new Vector2(0.375f, 25.0f));
 vertices[5] = new MyOwnVertexFormat(new Vector3(2, 1, -100), new Vector2(0.375f, 0.0f));
 vertices[6] = new MyOwnVertexFormat(new Vector3(3, 1, 10), new Vector2(0.5f, 25.0f));
 vertices[7] = new MyOwnVertexFormat(new Vector3(3, 1, -100), new Vector2(0.5f, 0.0f));
 vertices[8] = new MyOwnVertexFormat(new Vector3(13, 1, 10), new Vector2(0.75f, 25.0f));
 vertices[9] = new MyOwnVertexFormat(new Vector3(13, 1, -100), new Vector2(0.75f, 0.0f));
 vertices[10] = new MyOwnVertexFormat(new Vector3(13, 21, 10), new Vector2(1.25f, 25.0f));
 vertices[11] = new MyOwnVertexFormat(new Vector3(13, 21, -100), new Vector2(1.25f, 0.0f));
```

As you can see, only 12 vertices are needed to define 10 triangles. You can notice I have used horizontal texture coordinates which are outside the [0,1] range, such as -0.25f and 1.25f. Because in our shader we set the AdressU and AdressV states to Mirror, these points are mapped to 0.25f and 0.75f respectively, which creates a mirrored view, as you can see in the image below. The same trick was used with the vertical coordinates, where the texture was mirrored 25 times! If we wouldn’t have mirrored the texture image, that small image would have been stretched over our whole street. You can find more info on mirrored texture coordinates in this forum thread.

![Road Texture](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-08TriangleStrip3.jpg?raw=true)

Nothing has changed to the kind of information contained in our vertices: we’re still sending position and texture information for each vertex. So there’s no need to change the VertexDeclaration.

Before drawing, we’ll change the camera position, to get a nicer view. So change the contents of the SetUpCamera method to this:

```csharp
 private void SetUpCamera()
 {
     cameraPos = new Vector3(-25, 13, 18);
     viewMatrix = Matrix.CreateLookAt(cameraPos, new Vector3(0, 2, -12), new Vector3(0, 1, 0));
     projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 200.0f);
 }
```

I’ve also chosen black as my background color, I guess you know how to change this. All that’s left to do, is specify in the Draw method the amount of triangles we want to be drawn (10), and that we’ve defined them in a strip of triangles, instead of a list of triangles:

```csharp
 device.DrawPrimitives(PrimitiveType.TriangleStrip, 0, 10);
```

Where we declare we’ll be drawing a TriangleStrip, made out of 10 triangles.

That’s it! This chapter you’ve learned another way to help you reduce bandwidth.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-08TriangleStrip4.jpg?raw=true)

## Code so far

The HLSL has remained unchanged, so I’m only going to list the XNA code:

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


            streetTexture = Content.Load<Texture2D> ("streettexture");        }

        private void SetUpVertices()
        {

             MyOwnVertexFormat[] vertices = new MyOwnVertexFormat[12];
 
             vertices[0] = new MyOwnVertexFormat(new Vector3(-20, 0, 10), new Vector2(-0.25f, 25.0f));
             vertices[1] = new MyOwnVertexFormat(new Vector3(-20, 0, -100), new Vector2(-0.25f, 0.0f));
             vertices[2] = new MyOwnVertexFormat(new Vector3(2, 0, 10), new Vector2(0.25f, 25.0f));
             vertices[3] = new MyOwnVertexFormat(new Vector3(2, 0, -100), new Vector2(0.25f, 0.0f));
             vertices[4] = new MyOwnVertexFormat(new Vector3(2, 1, 10), new Vector2(0.375f, 25.0f));
             vertices[5] = new MyOwnVertexFormat(new Vector3(2, 1, -100), new Vector2(0.375f, 0.0f));
             vertices[6] = new MyOwnVertexFormat(new Vector3(3, 1, 10), new Vector2(0.5f, 25.0f));
             vertices[7] = new MyOwnVertexFormat(new Vector3(3, 1, -100), new Vector2(0.5f, 0.0f));
             vertices[8] = new MyOwnVertexFormat(new Vector3(13, 1, 10), new Vector2(0.75f, 25.0f));
             vertices[9] = new MyOwnVertexFormat(new Vector3(13, 1, -100), new Vector2(0.75f, 0.0f));
             vertices[10] = new MyOwnVertexFormat(new Vector3(13, 21, 10), new Vector2(1.25f, 25.0f));
             vertices[11] = new MyOwnVertexFormat(new Vector3(13, 21, -100), new Vector2(1.25f, 0.0f));
 
             vertexBuffer = new VertexBuffer(device, MyOwnVertexFormat.VertexDeclaration, vertices.Length, BufferUsage.WriteOnly);
             vertexBuffer.SetData(vertices);
         }
 
         private void SetUpCamera()
         {
             cameraPos = new Vector3(-25, 13, 18);
             viewMatrix = Matrix.CreateLookAt(cameraPos, new Vector3(0, 2, -12), new Vector3(0, 1, 0));
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
                 device.DrawPrimitives(PrimitiveType.TriangleStrip, 0, 10);
             }
 
             base.Draw(gameTime);
         }
     }
 }
```

## Next Steps

[World transformation](Riemers3DXNA3hlsl09worldtransform)
