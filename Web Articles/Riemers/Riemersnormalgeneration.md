# Automatically generating normals

When you’re reading this chapter, you probably already know why we need our vertices to contain normals. If you don’t you can have a look at this [chapter on lighting](Riemers3DXNA1Terrain11lighting) to get an idea.

However, when you define vertices yourself, or sometimes when you import them from a model, they don’t contain this normal data. So how can we create normals? For the mathematical background behind this chapter, you can have a look at the chapter in the ExtraReading section. This page contains the code.

So what we want is the ‘averaged normal in every vertex. If each vertex would be used for only one triangle, things would be very simple: we would simply need to store the normal of the triangle in each of its 4 vertices. However, if a vertex is shared by more than 1 triangle, we need to store the average of the normals of the triangles in our vertex.

The code I will show here starts from a vertexbuffer and an indexbuffer, which we will first convert to arrays of vertices and indices. So, if you have these arrays already at your disposal, you can simply leave out the first lines.

We’ll create a method, which generates the normals. It will start from a vertex- and indexbuffer:

```csharp
 private void GenerateNormals(VertexBuffer vb, IndexBuffer ib)
 {
     VertexPositionNormalColored[] vertices = new VertexPositionNormalColored[WIDTH * HEIGHT];
     vb.GetData(vertices);
     int[] indices = new int[(WIDTH - 1) * (HEIGHT - 1) * 6];
     ib.GetData(indices);
 }
```

So the method takes in the vertex- and indexbuffer, and converts them to arrays. While doing this, we need to indicate what type of vertices are contained in the vertex buffer, and how big our arrays must be. The values taken in this example correspond to the final chapter of Series 1.

The first step would be to reset all normals in the vertex array. This is easy to do:

```csharp
 for (int i = 0; i < vertices.Length; i++)
      vertices[i].Normal = new Vector3(0, 0, 0);
```

Next comes the main part of the method: calculating the normal. For each triangle, we calculate the normal. Then we add this normal to each of the 3 vertices of that triangle. This is how it’s done:

```csharp
 for (int i = 0; i < indices.Length / 3; i++)
 {
      Vector3 firstvec = vertices[indices[i*3+1]].Position-vertices[indices[i*3]].Position;
      Vector3 secondvec = vertices[indices[i*3]].Position-vertices[indices[i*3+2]].Position;
      Vector3 normal = Vector3.Cross(firstvec, secondvec);
      normal.Normalize();
      vertices[indices[i * 3]].Normal += normal;
      vertices[indices[i * 3 + 1]].Normal += normal;
      vertices[indices[i * 3 + 2]].Normal += normal;
 }
```

The indexbuffer contains the order in which vertices are used to create the triangles. Because our indexbuffer describes a trianglelist, the number of triangles equals the length of our indexbuffer divided by 3.

For each triangle, we calculate its normal, by taking the cross product of the 2 planar vectors, and normalizing the result (this is explained in more detail in the ExtreReading section). Note: a ‘normal’ is the vector perpendicular to the triangle, while ‘normalizing a vector’ makes the length exactly 1.

When we have the normal of the triangle, we add it to the normals of the 3 vertices of that triangle. When we do this for each triangle, the resulting normals in each vertex will have the correct direction, but they will be too large, so we need to normalize again:

```csharp
 for (int i = 0; i < vertices.Length; i++)
     vertices[i].Normal.Normalize();
```

That’s it! Now each vertex our vertex array contains a normal, which is the average of the normals of all triangles using that vertex. In the case you want to update your vertex-and indexbuffers, you’ll only have to update your vertexbuffer, as nothing has changed to the indices:

```csharp
 vb.SetData(vertices);
```

This concludes the code on normal generation. As a sidenote, I should mention that if you have created your arrays from buffers, these buffers must not have been created with the ResourceUsage.WriteOnly flag specified, because in the first lines of our code we’re reading data from our buffers.

![Normals](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersNormals1.jpg?raw=true)

Here’s the code, with the method integrated into the terrain of our first series:

```csharp
 using System;
 using System.Collections.Generic;
 using Microsoft.Xna.Framework;
 using Microsoft.Xna.Framework.Audio;
 using Microsoft.Xna.Framework.Content;
 using Microsoft.Xna.Framework.Graphics;
 using Microsoft.Xna.Framework.Input;
 using Microsoft.Xna.Framework.Storage;
 using System.IO;
 
 namespace XNAtutorial
 {
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         public struct VertexPositionNormalColored
         {
             public Vector3 Position;
             public Color Color;
             public Vector3 Normal;
 
             public static int SizeInBytes = 7 * 4;
             public static VertexElement[] VertexElements = new VertexElement[]
              {
                  new VertexElement( 0, 0, VertexElementFormat.Vector3, VertexElementMethod.Default, VertexElementUsage.Position, 0 ),
                  new VertexElement( 0, sizeof(float) * 3, VertexElementFormat.Color, VertexElementMethod.Default, VertexElementUsage.Color, 0 ),
                  new VertexElement( 0, sizeof(float) * 4, VertexElementFormat.Vector3, VertexElementMethod.Default, VertexElementUsage.Normal, 0 ),
              };
         }
 
         GraphicsDeviceManager graphics;
         ContentManager content;
         GraphicsDevice device;
         Effect effect;
         VertexPositionNormalColored[] vertices;
         private float angle = 0f;
         private VertexBuffer vb;
         private IndexBuffer ib;
         private int WIDTH = 64;
         private int HEIGHT = 64;
         private int[,] heightData;
 
         private int MinimumHeight = 255;
         private int MaximumHeight = 0;
 
 
         public Game1()
         {
             graphics = new GraphicsDeviceManager(this);
             content = new ContentManager(Services);
         }
 
         protected override void Initialize()
         {
             base.Initialize();
             LoadHeightData();
             SetUpXNADevice();
             SetUpVertices();
             SetUpIndices();
             GenerateNormals(vb, ib);
             SetUpCamera();
         }
 
         private void SetUpXNADevice()
         {
             device = graphics.GraphicsDevice;
 
             graphics.PreferredBackBufferWidth = 500;
             graphics.PreferredBackBufferHeight = 500;
             graphics.IsFullScreen = false;
             graphics.ApplyChanges();
             Window.Title = "Riemer's XNA Tutorials -- Series 1";
 
             CompiledEffect compiledEffect = Effect.CompileEffectFromFile("@/../../../../effects.fx", null, null, CompilerOptions.None, TargetPlatform.Windows);
             effect = new Effect(graphics.GraphicsDevice, compiledEffect.GetEffectCode(), CompilerOptions.None, null);
         }
 
         private void SetUpVertices()
         {
 
             vertices = new VertexPositionNormalColored[WIDTH * HEIGHT];
 
             for (int x = 0; x < WIDTH; x++)
             {
                 for (int y = 0; y < HEIGHT; y++)
                 {
                     vertices[x + y * WIDTH].Position = new Vector3(x, y, heightData[x, y]);
 
                     vertices[x + y * WIDTH].Normal = new Vector3(0, 0, 1);
 
 
                     if (heightData[x, y] < MinimumHeight + (MaximumHeight - MinimumHeight) / 4)
                     {
                         vertices[x + y * WIDTH].Color = Color.Blue;
                     }
                     else if (heightData[x, y] < MinimumHeight + (MaximumHeight - MinimumHeight) * 2 / 4)
                     {
                         vertices[x + y * WIDTH].Color = Color.Green;
                     }
                     else if (heightData[x, y] < MinimumHeight + (MaximumHeight - MinimumHeight) * 3 / 4)
                     {
                         vertices[x + y * WIDTH].Color = Color.Brown;
                     }
                     else
                     {
                         vertices[x + y * WIDTH].Color = Color.White;
                     }
                 }
             }
 
             vb = new VertexBuffer(device, VertexPositionNormalColored.SizeInBytes * WIDTH * HEIGHT, ResourceUsage.None, ResourceManagementMode.Automatic);
             vb.SetData(vertices);
         }
 
         private void SetUpIndices()
         {
             int[] indices = new int[(WIDTH - 1) * (HEIGHT - 1) * 6];
             for (int x = 0; x < WIDTH - 1; x++)
             {
                 for (int y = 0; y < HEIGHT - 1; y++)
                 {
                     indices[(x + y * (WIDTH - 1)) * 6] = (x + 1) + (y + 1) * WIDTH;
                     indices[(x + y * (WIDTH - 1)) * 6 + 1] = (x + 1) + y * WIDTH;
                     indices[(x + y * (WIDTH - 1)) * 6 + 2] = x + y * WIDTH;
 
                     indices[(x + y * (WIDTH - 1)) * 6 + 3] = (x + 1) + (y + 1) * WIDTH;
                     indices[(x + y * (WIDTH - 1)) * 6 + 4] = x + y * WIDTH;
                     indices[(x + y * (WIDTH - 1)) * 6 + 5] = x + (y + 1) * WIDTH;
                 }
             }
 
             ib = new IndexBuffer(device, typeof(int), (WIDTH - 1) * (HEIGHT - 1) * 6, ResourceUsage.None, ResourceManagementMode.Automatic);
             ib.SetData(indices);
         }
 
         private void GenerateNormals(VertexBuffer vb, IndexBuffer ib)
         {
             VertexPositionNormalColored[] vertices = new VertexPositionNormalColored[WIDTH * HEIGHT];
             vb.GetData(vertices);
             int[] indices = new int[(WIDTH - 1) * (HEIGHT - 1) * 6];
             ib.GetData(indices);
 
             for (int i = 0; i < vertices.Length; i++)
                 vertices[i].Normal = new Vector3(0, 0, 0);
 
             for (int i = 0; i < indices.Length / 3; i++)
             {
                 Vector3 firstvec = vertices[indices[i*3+1]].Position-vertices[indices[i*3]].Position;
                 Vector3 secondvec = vertices[indices[i*3]].Position-vertices[indices[i*3+2]].Position;
                 Vector3 normal = Vector3.Cross(firstvec, secondvec);
                 normal.Normalize();
                 vertices[indices[i * 3]].Normal += normal;
                 vertices[indices[i * 3 + 1]].Normal += normal;
                 vertices[indices[i * 3 + 2]].Normal += normal;
             }
 
             for (int i = 0; i < vertices.Length; i++)
                 vertices[i].Normal.Normalize();
 
             vb.SetData(vertices);
         }
 
         private void SetUpCamera()
         {
             Matrix viewMatrix = Matrix.CreateLookAt(new Vector3(80, 0, 160), new Vector3(-20, 0, 0), new Vector3(0, 0, 1));
             Matrix projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, this.Window.ClientBounds.Width / this.Window.ClientBounds.Height, 1.0f, 250.0f);
 
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
         }
 
         private void LoadHeightData()
         {
             int offset;
             FileStream fs = new FileStream("../../../heightmap.bmp", FileMode.Open, FileAccess.Read);
             BinaryReader r = new BinaryReader(fs);
 
             r.BaseStream.Seek(10, SeekOrigin.Current);
             offset = (int)r.ReadUInt32();
 
             r.BaseStream.Seek(4, SeekOrigin.Current);
             WIDTH = (int)r.ReadUInt32();
             HEIGHT = (int)r.ReadUInt32();
 
             r.BaseStream.Seek(offset - 26, SeekOrigin.Current);
             heightData = new int[WIDTH, HEIGHT];
             for (int i = 0; i < HEIGHT; i++)
             {
                 for (int y = 0; y < WIDTH; y++)
                 {
                     int height = (int)(r.ReadByte());
                     height += (int)(r.ReadByte());
                     height += (int)(r.ReadByte());
                     height /= 8;
                     heightData[WIDTH - 1 - y, HEIGHT - 1 - i] = height;
 
                     if (height < MinimumHeight)
                     {
                         MinimumHeight = height;
                     }
                     if (height > MaximumHeight)
                     {
                         MaximumHeight = height;
                     }
 
                 }
             }
         }
 
         protected override void LoadGraphicsContent(bool loadAllContent)
         {
             if (loadAllContent)
             {
             }
         }
 
         protected override void UnloadGraphicsContent(bool unloadAllContent)
         {
             if (unloadAllContent == true)
             {
                 content.Unload();
             }
         }
 
         protected override void Update(GameTime gameTime)
         {
             if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                 this.Exit();
             base.Update(gameTime);
 
             ProcessKeyboard();
         }
 
         private void ProcessKeyboard()
         {
             KeyboardState keys = Keyboard.GetState();
             if (keys.IsKeyDown(Keys.Delete))
             {
                 angle += 0.05f;
             }
             if (keys.IsKeyDown(Keys.PageDown))
             {
                 angle -= 0.05f;
             }
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.RenderState.FillMode = FillMode.Solid;
             device.RenderState.CullMode = CullMode.None;
 
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
 
             effect.CurrentTechnique = effect.Techniques["Colored"];
             Matrix worldMatrix = Matrix.CreateTranslation(-HEIGHT / 2, -WIDTH / 2, 0) * Matrix.CreateRotationZ(angle); ;
             effect.Parameters["xWorld"].SetValue(worldMatrix);
 
             effect.Parameters["xEnableLighting"].SetValue(true);
             effect.Parameters["xLightDirection"].SetValue(new Vector3(0f, 0.4f, -0.7f));
 
             effect.Begin();
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Begin();
 
 
                 device.Vertices[0].SetSource(vb, 0, VertexPositionNormalColored.SizeInBytes);
                 device.Indices = ib;
                 device.VertexDeclaration = new VertexDeclaration(device, VertexPositionNormalColored.VertexElements);
 
                 device.DrawIndexedPrimitives(PrimitiveType.TriangleList, 0, 0, WIDTH * HEIGHT, 0, (WIDTH - 1) * (HEIGHT - 1) * 2);
 
                 pass.End();
             }
             effect.End();
 
             base.Draw(gameTime);
         }
     }
 }
```
