# Texturing our terrain

As we’ve already seen quite some topics on texturing triangles, I’ll go over the first part on terrain texturing rather quickly. We’ll cover our complete terrain by a grass texture, which will be copied a few times over our terrain in mirrored mode. Of course, we’ll need a grass texture for this, which you can download here. Import it into your project, add this variable:

```csharp
 Texture2D grassTexture;
```

Let’s create a small separate method to load all our textures:

```csharp
 private void LoadTextures()
 {

    grassTexture = Content.Load<Texture2D> ("grass");}
```

And call this method from the end of our LoadContent method:

```csharp
 LoadTextures();
```

Instead of using our custom defined VertexPositionNormalColored format, we’ll do with the default VertexPositionNormalTexture format. Don’t delete the struct though, as we’ll adjust it in the next chapter. However, we’ll have to replace all instances of VertexPositionNormalColored in our code to VertexPositionNormalTexture. You’ll want to use Ctrl+H for this. Warning: don’t change the name of your struct into VertexPositionNormalTexture, as this will override XNA’s VertexPositionNormalTexture struct!

Instead of specifying the color of each vertex, we’ll need to pass in the texture coordinate. So this is the new SetUpTerrainVertices method:

```csharp
 private VertexPositionNormalTexture[] SetUpTerrainVertices()
 {
     VertexPositionNormalTexture[] terrainVertices = new VertexPositionNormalTexture[terrainWidth * terrainLength];

     for (int x = 0; x < terrainWidth; x++)
     {
         for (int y = 0; y < terrainLength; y++)
         {
             terrainVertices[x + y * terrainWidth].Position = new Vector3(x, heightData[x, y], -y);
             terrainVertices[x + y * terrainWidth].TextureCoordinate.X = (float)x / 30.0f;
             terrainVertices[x + y * terrainWidth].TextureCoordinate.Y = (float)y / 30.0f;
         }
     }

     return terrainVertices;
 }
```

In the Draw method, we need to indicate that we’ll be using the Textured technique to draw our terrain, instead of the Colored technique:

```csharp
 effect.CurrentTechnique = effect.Techniques["Textured"];
 effect.Parameters["xTexture"].SetValue(grassTexture);
```

You see we’ve also passed in our grass texture.

That should be it! When you run this code, you should see a terrain nicely covered with grass.

![Textured Terrain](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-03Terrain1.jpg?raw=true)

> You can try these exercises to practice what you've learned:
>
> - Try changing the division factor in the SetUpTerrainVertices method. Make sure you try some crazily big and large numbers.

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
 
 namespace XNAseries4
 {
     public struct VertexPositionNormalColor
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
 
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         GraphicsDeviceManager graphics;
         GraphicsDevice device;
 
         int terrainWidth;
         int terrainLength;
         float[,] heightData;
 
         VertexBuffer terrainVertexBuffer;
         IndexBuffer terrainIndexBuffer;
         VertexDeclaration terrainVertexDeclaration;
 
         Effect effect;
         Matrix viewMatrix;
         Matrix projectionMatrix;
 
         Vector3 cameraPosition = new Vector3(130, 30, -50);
         float leftrightRot = MathHelper.PiOver2;
         float updownRot = -MathHelper.Pi / 10.0f;
         const float rotationSpeed = 0.3f;
         const float moveSpeed = 30.0f;
         MouseState originalMouseState;
 
         Texture2D grassTexture;
 
         public Game1()
         {
             graphics = new GraphicsDeviceManager(this);
             Content.RootDirectory = "Content";
         }
 
         protected override void Initialize()
         {
             graphics.PreferredBackBufferWidth = 500;
             graphics.PreferredBackBufferHeight = 500;
 
             graphics.ApplyChanges();
             Window.Title = "Riemer's XNA Tutorials -- Series 4";
 
             base.Initialize();
         }
 
         protected override void LoadContent()
         {
             device = GraphicsDevice;

            effect = Content.Load<Effect> ("Series4Effects");
            UpdateViewMatrix();
            projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 0.3f, 1000.0f);

            Mouse.SetPosition(device.Viewport.Width / 2, device.Viewport.Height / 2);
            originalMouseState = Mouse.GetState();

            LoadVertices();

             LoadTextures();
         }
 
         private void LoadVertices()
         {

            Texture2D heightMap = Content.Load<Texture2D> ("heightmap");            LoadHeightData(heightMap);


             VertexPositionNormalTexture[] terrainVertices = SetUpTerrainVertices();
             int[] terrainIndices = SetUpTerrainIndices();
             terrainVertices = CalculateNormals(terrainVertices, terrainIndices);
             CopyToTerrainBuffers(terrainVertices, terrainIndices);
             terrainVertexDeclaration = new VertexDeclaration(device, VertexPositionNormalTexture.VertexElements);
         }
 
         private void LoadTextures()
         {

            grassTexture = Content.Load<Texture2D> ("grass");        }

 
         private void LoadHeightData(Texture2D heightMap)
         {
             float minimumHeight = float.MaxValue;
             float maximumHeight = float.MinValue;
 
             terrainWidth = heightMap.Width;
             terrainLength = heightMap.Height;
 
             Color[] heightMapColors = new Color[terrainWidth * terrainLength];
             heightMap.GetData(heightMapColors);
 
             heightData = new float[terrainWidth, terrainLength];
             for (int x = 0; x < terrainWidth; x++)
                 for (int y = 0; y < terrainLength; y++)
                 {
                     heightData[x, y] = heightMapColors[x + y * terrainWidth].R;
                     if (heightData[x, y] < minimumHeight) minimumHeight = heightData[x, y];
                     if (heightData[x, y] > maximumHeight) maximumHeight = heightData[x, y];
                 }
 
             for (int x = 0; x < terrainWidth; x++)
                 for (int y = 0; y < terrainLength; y++)
                     heightData[x, y] = (heightData[x, y] - minimumHeight) / (maximumHeight - minimumHeight) * 30.0f;
         }
 
         private VertexPositionNormalTexture[] SetUpTerrainVertices()
         {
             VertexPositionNormalTexture[] terrainVertices = new VertexPositionNormalTexture[terrainWidth * terrainLength];
 
             for (int x = 0; x < terrainWidth; x++)
             {
                 for (int y = 0; y < terrainLength; y++)
                 {
                     terrainVertices[x + y * terrainWidth].Position = new Vector3(x, heightData[x, y], -y);
                     terrainVertices[x + y * terrainWidth].TextureCoordinate.X = (float)x / 30.0f;
                     terrainVertices[x + y * terrainWidth].TextureCoordinate.Y = (float)y / 30.0f;
                 }
             }
 
             return terrainVertices;
         }
 
         private int[] SetUpTerrainIndices()
         {
             int[] indices = new int[(terrainWidth - 1) * (terrainLength - 1) * 6];
             int counter = 0;
             for (int y = 0; y < terrainLength - 1; y++)
             {
                 for (int x = 0; x < terrainWidth - 1; x++)
                 {
                     int lowerLeft = x + y * terrainWidth;
                     int lowerRight = (x + 1) + y * terrainWidth;
                     int topLeft = x + (y + 1) * terrainWidth;
                     int topRight = (x + 1) + (y + 1) * terrainWidth;
 
                     indices[counter++] = topLeft;
                     indices[counter++] = lowerRight;
                     indices[counter++] = lowerLeft;
 
                     indices[counter++] = topLeft;
                     indices[counter++] = topRight;
                     indices[counter++] = lowerRight;
                 }
             }
 
             return indices;
         }
 
         private VertexPositionNormalTexture[] CalculateNormals(VertexPositionNormalTexture[] vertices, int[] indices)
         {
             for (int i = 0; i < vertices.Length; i++)
                 vertices[i].Normal = new Vector3(0, 0, 0);
 
             for (int i = 0; i < indices.Length / 3; i++)
             {
                 int index1 = indices[i * 3];
                 int index2 = indices[i * 3 + 1];
                 int index3 = indices[i * 3 + 2];
 
                 Vector3 side1 = vertices[index1].Position - vertices[index3].Position;
                 Vector3 side2 = vertices[index1].Position - vertices[index2].Position;
                 Vector3 normal = Vector3.Cross(side1, side2);
 
                 vertices[index1].Normal += normal;
                 vertices[index2].Normal += normal;
                 vertices[index3].Normal += normal;
             }
 
             for (int i = 0; i < vertices.Length; i++)
                 vertices[i].Normal.Normalize();
 
             return vertices;
         }
 
         private void CopyToTerrainBuffers(VertexPositionNormalTexture[] vertices, int[] indices)
         {
             terrainVertexBuffer = new VertexBuffer(device, vertices.Length * VertexPositionNormalTexture.SizeInBytes, BufferUsage.WriteOnly);
             terrainVertexBuffer.SetData(vertices);
 
             terrainIndexBuffer = new IndexBuffer(device, typeof(int), indices.Length, BufferUsage.WriteOnly);
             terrainIndexBuffer.SetData(indices);
         }
 
         protected override void UnloadContent()
         {
         }
 
         protected override void Update(GameTime gameTime)
         {
             if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                 this.Exit();
 
             float timeDifference = (float)gameTime.ElapsedGameTime.TotalMilliseconds / 1000.0f;
             ProcessInput(timeDifference);
 
             base.Update(gameTime);
         }
 
         private void ProcessInput(float amount)
         {
             MouseState currentMouseState = Mouse.GetState();
             if (currentMouseState != originalMouseState)
             {
                 float xDifference = currentMouseState.X - originalMouseState.X;
                 float yDifference = currentMouseState.Y - originalMouseState.Y;
                 leftrightRot -= rotationSpeed * xDifference * amount;
                 updownRot -= rotationSpeed * yDifference * amount;
                 Mouse.SetPosition(device.Viewport.Width / 2, device.Viewport.Height / 2);
                 UpdateViewMatrix();
             }
 
             Vector3 moveVector = new Vector3(0, 0, 0);
             KeyboardState keyState = Keyboard.GetState();
             if (keyState.IsKeyDown(Keys.Up) || keyState.IsKeyDown(Keys.W))
                 moveVector += new Vector3(0, 0, -1);
             if (keyState.IsKeyDown(Keys.Down) || keyState.IsKeyDown(Keys.S))
                 moveVector += new Vector3(0, 0, 1);
             if (keyState.IsKeyDown(Keys.Right) || keyState.IsKeyDown(Keys.D))
                 moveVector += new Vector3(1, 0, 0);
             if (keyState.IsKeyDown(Keys.Left) || keyState.IsKeyDown(Keys.A))
                 moveVector += new Vector3(-1, 0, 0);
             if (keyState.IsKeyDown(Keys.Q))
                 moveVector += new Vector3(0, 1, 0);
             if (keyState.IsKeyDown(Keys.Z))
                 moveVector += new Vector3(0, -1, 0);
             AddToCameraPosition(moveVector * amount);
         }
 
         private void AddToCameraPosition(Vector3 vectorToAdd)
         {
             Matrix cameraRotation = Matrix.CreateRotationX(updownRot) * Matrix.CreateRotationY(leftrightRot);
             Vector3 rotatedVector = Vector3.Transform(vectorToAdd, cameraRotation);
             cameraPosition += moveSpeed * rotatedVector;
             UpdateViewMatrix();
         }
 
         private void UpdateViewMatrix()
         {
             Matrix cameraRotation = Matrix.CreateRotationX(updownRot) * Matrix.CreateRotationY(leftrightRot);
 
             Vector3 cameraOriginalTarget = new Vector3(0, 0, -1);
             Vector3 cameraOriginalUpVector = new Vector3(0, 1, 0);
 
             Vector3 cameraRotatedTarget = Vector3.Transform(cameraOriginalTarget, cameraRotation);
             Vector3 cameraFinalTarget = cameraPosition + cameraRotatedTarget;
 
             Vector3 cameraRotatedUpVector = Vector3.Transform(cameraOriginalUpVector, cameraRotation);
 
             viewMatrix = Matrix.CreateLookAt(cameraPosition, cameraFinalTarget, cameraRotatedUpVector);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             float time = (float)gameTime.TotalGameTime.TotalMilliseconds / 100.0f;
             device.RenderState.CullMode = CullMode.None;
 
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
             DrawTerrain(viewMatrix);
 
             base.Draw(gameTime);
         }
 
         private void DrawTerrain(Matrix currentViewMatrix)
         {
             effect.CurrentTechnique = effect.Techniques["Textured"];
             effect.Parameters["xTexture"].SetValue(grassTexture);
 
             Matrix worldMatrix = Matrix.Identity;
             effect.Parameters["xWorld"].SetValue(worldMatrix);
             effect.Parameters["xView"].SetValue(currentViewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);            
 
             effect.Parameters["xEnableLighting"].SetValue(true);
             effect.Parameters["xAmbient"].SetValue(0.4f);
             effect.Parameters["xLightDirection"].SetValue(new Vector3(-0.5f, -1, -0.5f));
 
             effect.Begin();
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Begin();
 
                 device.Vertices[0].SetSource(terrainVertexBuffer, 0, VertexPositionNormalTexture.SizeInBytes);
                 device.Indices = terrainIndexBuffer;
                 device.VertexDeclaration = terrainVertexDeclaration;
 
                 int noVertices = terrainVertexBuffer.SizeInBytes / VertexPositionNormalTexture.SizeInBytes;
                 int noTriangles = terrainIndexBuffer.SizeInBytes / sizeof(int) / 3;
                 device.DrawIndexedPrimitives(PrimitiveType.TriangleList, 0, 0, noVertices, 0, noTriangles);
 
                 pass.End();
             }
             effect.End();
         }
     }
 }
```

## Next Steps

[Multitexturing](Riemers3DXNA4advterrain04multitexturing)
