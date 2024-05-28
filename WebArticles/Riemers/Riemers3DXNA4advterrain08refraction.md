# Rendering the refractive map –- XNA Clip planes

As a first step towards drawing our water, we’ll start with rendering the refractive map into a texture. We’ll need this map for each pixel of our water, to determine the color of what’s underneath that pixel.

In fact, this almost comes down to rendering the scene, as we see it through the camera, into a texture. However, there are cases where some hills could obstruct our view to the bottom of the river behind them. This would mean that for the pixels of that river, we would sample the color of the hill, instead of the bottom of the river.

As we’ll be rendering to a texture, we’ll need these variables:

```csharp
 const float waterHeight = 5.0f;
 RenderTarget2D refractionRenderTarget;
 Texture2D refractionMap;
```

The first line will indicate how high we want our water to be positioned.

We need to initialize the rendertarget, so add this code to our LoadContent method:

```csharp
 PresentationParameters pp = device.PresentationParameters;
 refractionRenderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, 1, device.DisplayMode.Format);
```

So, we only want to draw the things that are BELOW the water, and clip everything above the water away. To do this, we can set a user clip plane. The name explains what it does: it is a plane, and the part of the scene on one side of that plane is clipped away. In our case, we will define a horizontal plane at the height of the water, so everything above that plane will not be drawn.

Defining a plane can get pretty mathematical. The easiest way to define a plane is to specify the normal direction of the plane, and its shortest distance to the (0,0,0) point of the scene. In our case, we’re in luck because our plane is completely horizontal: in that case the normal is simply the (0,1,0) Up-vector, and the distance to the (0,0,0) point is the height level of our water.

Because the lowest points of our terrain have Y height coordinate 0, we cannot define the water at Y coordinate 0, as the water would be lower than all vertices in our terrain. Instead, we’ll say the water is at Y coordinate 5. This will also bring us to a more general approach, as a plane through the (0,0,0) is an exceptional case.

As a summary: we need to define a horizontal plane at height Y=5, which we’ll use as a clip plane, so everything above this plane will not be drawn.

We’ll start by creating the plane. As I said, we need the height, and the normal direction. You can easily see this is enough to create a plane: the normal implies the “rotation” of the plane, while the height indicates the distance to the (0,0,0) point. For each combination of normal and distance, there is only one plane.

There remains only one thing we need to specify: whether we want to clip everything away below or above the plane. We’ll pass this to the method as a Boolean:

```csharp
 private Plane CreatePlane(float height, Vector3 planeNormalDirection, Matrix currentViewMatrix, bool clipSide)
 {
     planeNormalDirection.Normalize();
     Vector4 planeCoeffs = new Vector4(planeNormalDirection, height);
     if (clipSide)
     planeCoeffs *= -1;
 }
```

First we make sure our normal is of unity length. Next, we create the plane coefficients, from which XNA can create a Plane. It is important to note that the line above would give exactly the same plane as:

```csharp
 Vector4 planeCoeffs = new Vector4(-planeNormalDirection, -height);
```

The difference is that in the first plane, the world below the plane will be clipped away, while in the latter the stuff above the plane will be clipped away.

This is what we use in the last 2 lines: if we specify clipSide=false, we want to clip everything away below the plane. If we want the clip everything away above the plane, we will specify clipSide=true, which results in a sign change of the 4 coefficients.

From this Vector4 we could already create a Plane. There remains, however, one huge pitfall when creating a clip plane, for which you better take a deep breath before you read on. As the clipping will occur in hardware after they have passed the vertex shader, the vertices which will be compared to the plane will already be in camera space coordinates. This means we need to transform our plane coefficients with the inverse of the camera matrix, before creating the plane from the coefficients. Remember, this camera matrix is the combination of the world,view and projection matrix.

We get the inverse of a matrix by inverting it and taking the transpose of the inverted matrix. So this is what we end up with:

```csharp
 Matrix worldViewProjection = currentViewMatrix * projectionMatrix;
 Matrix inverseWorldViewProjection = Matrix.Invert(worldViewProjection);
 inverseWorldViewProjection = Matrix.Transpose(inverseWorldViewProjection);
```

Note that the vertices of the terrain and water are already defined in absolute World space, so we use Matrix.Identity as World matrix, or even better simply leave it away.

Now we can transform our plane coefficients into the correct space, and create our plane :

```csharp
 planeCoeffs = Vector4.Transform(planeCoeffs, inverseWorldViewProjection);
 Plane finalPlane = new Plane(planeCoeffs);

 return finalPlane;
```

With our plane ready, we are ready to define a method DrawRefractionMap, which, well, draws the refraction map:

```csharp
 private void DrawRefractionMap()
 {
     Plane refractionPlane = CreatePlane(waterHeight + 1.5f, new Vector3(0,-1,0), viewMatrix, false);
     device.ClipPlanes[0].Plane = refractionPlane;
     device.ClipPlanes[0].IsEnabled = true;
     device.SetRenderTarget(0, refractionRenderTarget);
     device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
     DrawTerrain(viewMatrix);
     device.ClipPlanes[0].IsEnabled = false;

     device.SetRenderTarget(0, null);
     refractionMap = refractionRenderTarget.GetTexture();
 }
```

The first line creates a horizontal plane, a bit above the value of the waterHeight variable. The last argument indicates that we only want the things under the plane to be rendered. The second and third lines activate the clipping plane.

Once the clip plane is enabled, you set the custom render target as current render target and you clean it. Next, you render the terrain onto the render target (only the part below the clip plane will be rendered!), after which you disable the clip plane and store the contents of the render target into the refractionMap texture.

Make sure you call this method at the beginning of our Draw method:

```csharp
 DrawRefractionMap();
```

That’s it for this chapter! When you run this code, you should get the same result as last chapter, but in the background the refraction map is stored into an image. Try putting this line at the end of the DrawRefractionMap method:

```csharp
 refractionMap.Save("refractionmap.jpg", ImageFileFormat.Jpg);
```

This will dump the texture to a file, so you can verily everything is running ok. Don’t forget to remove this line before going to the next chapter, as this will lower your framerate a lot. Anyway, this is what should be in the refraction map:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-08Refaction1.jpg?raw=true)

You can clearly see this corresponds to the parts that are below the water. So now, for each pixel of the water, we already know what color is beneat it. Next is the reflection map, which needs the same clip plane, but the other side needs to be clipped.

> You can try these exercises to practice what you've learned:
>
> - Try changing the height of the clip plane.
> - Try clipping away the other side of the plane.
> - Try to clip everything away with X coordinate larger than 10.

## Our XNA code

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
     public struct VertexMultitextured
     {
         public Vector3 Position;
         public Vector3 Normal;
         public Vector4 TextureCoordinate;
         public Vector4 TexWeights;
 
         public static int SizeInBytes = (3 + 3 + 4 + 4) * sizeof(float);
         public static VertexElement[] VertexElements = new VertexElement[]
         {
             new VertexElement( 0, 0, VertexElementFormat.Vector3, VertexElementMethod.Default, VertexElementUsage.Position, 0 ),
             new VertexElement( 0, sizeof(float) * 3, VertexElementFormat.Vector3, VertexElementMethod.Default, VertexElementUsage.Normal, 0 ),
             new VertexElement( 0, sizeof(float) * 6, VertexElementFormat.Vector4, VertexElementMethod.Default, VertexElementUsage.TextureCoordinate, 0 ),
             new VertexElement( 0, sizeof(float) * 10, VertexElementFormat.Vector4, VertexElementMethod.Default, VertexElementUsage.TextureCoordinate, 1 ),
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
         Texture2D sandTexture;
         Texture2D rockTexture;
         Texture2D snowTexture;
         Texture2D cloudMap;
         
         Model skyDome;
 
         const float waterHeight = 5.0f;        
         RenderTarget2D refractionRenderTarget;
         Texture2D refractionMap;
 
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


            skyDome = Content.Load<Model> ("dome");            skyDome.Meshes[0].MeshParts[0].Effect = effect.Clone(device);


             PresentationParameters pp = device.PresentationParameters;
             refractionRenderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, 1, device.DisplayMode.Format);
 
             LoadVertices();
             LoadTextures();
         }
 
         private void LoadVertices()
         {

            Texture2D heightMap = Content.Load<Texture2D> ("heightmap");            LoadHeightData(heightMap);

            VertexMultitextured[] terrainVertices = SetUpTerrainVertices();
            int[] terrainIndices = SetUpTerrainIndices();
            terrainVertices = CalculateNormals(terrainVertices, terrainIndices);
            CopyToTerrainBuffers(terrainVertices, terrainIndices);
            terrainVertexDeclaration = new VertexDeclaration(device, VertexMultitextured.VertexElements);
        }

        private void LoadTextures()
        {

            grassTexture = Content.Load<Texture2D> ("grass");
            sandTexture = Content.Load<Texture2D> ("sand");
            rockTexture = Content.Load<Texture2D> ("rock");
            snowTexture = Content.Load<Texture2D> ("snow");
            cloudMap = Content.Load<Texture2D> ("cloudMap");        }

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

        private VertexMultitextured[] SetUpTerrainVertices()
        {
            VertexMultitextured[] terrainVertices = new VertexMultitextured[terrainWidth * terrainLength];

            for (int x = 0; x < terrainWidth; x++)
            {
                for (int y = 0; y < terrainLength; y++)
                {
                    terrainVertices[x + y * terrainWidth].Position = new Vector3(x, heightData[x, y], -y);
                    terrainVertices[x + y * terrainWidth].TextureCoordinate.X = (float)x / 30.0f;
                    terrainVertices[x + y * terrainWidth].TextureCoordinate.Y = (float)y / 30.0f;

                    terrainVertices[x + y * terrainWidth].TexWeights.X = MathHelper.Clamp(1.0f - Math.Abs(heightData[x, y] - 0) / 8.0f, 0, 1);
                    terrainVertices[x + y * terrainWidth].TexWeights.Y = MathHelper.Clamp(1.0f - Math.Abs(heightData[x, y] - 12) / 6.0f, 0, 1);
                    terrainVertices[x + y * terrainWidth].TexWeights.Z = MathHelper.Clamp(1.0f - Math.Abs(heightData[x, y] - 20) / 6.0f, 0, 1);
                    terrainVertices[x + y * terrainWidth].TexWeights.W = MathHelper.Clamp(1.0f - Math.Abs(heightData[x, y] - 30) / 6.0f, 0, 1);

                    float total = terrainVertices[x + y * terrainWidth].TexWeights.X;
                    total += terrainVertices[x + y * terrainWidth].TexWeights.Y;
                    total += terrainVertices[x + y * terrainWidth].TexWeights.Z;
                    total += terrainVertices[x + y * terrainWidth].TexWeights.W;

                    terrainVertices[x + y * terrainWidth].TexWeights.X /= total;
                    terrainVertices[x + y * terrainWidth].TexWeights.Y /= total;
                    terrainVertices[x + y * terrainWidth].TexWeights.Z /= total;
                    terrainVertices[x + y * terrainWidth].TexWeights.W /= total;
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

        private VertexMultitextured[] CalculateNormals(VertexMultitextured[] vertices, int[] indices)
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

        private void CopyToTerrainBuffers(VertexMultitextured[] vertices, int[] indices)
        {
            terrainVertexBuffer = new VertexBuffer(device, vertices.Length * VertexMultitextured.SizeInBytes, BufferUsage.WriteOnly);
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


             DrawRefractionMap();
 
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
             DrawSkyDome(viewMatrix);
             DrawTerrain(viewMatrix);            
 
             base.Draw(gameTime);
         }
 
         private void DrawTerrain(Matrix currentViewMatrix)
         {
             effect.CurrentTechnique = effect.Techniques["MultiTextured"];
             effect.Parameters["xTexture0"].SetValue(sandTexture);
             effect.Parameters["xTexture1"].SetValue(grassTexture);
             effect.Parameters["xTexture2"].SetValue(rockTexture);
             effect.Parameters["xTexture3"].SetValue(snowTexture);
 
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
 
                 device.Vertices[0].SetSource(terrainVertexBuffer, 0, VertexMultitextured.SizeInBytes);
                 device.Indices = terrainIndexBuffer;
                 device.VertexDeclaration = terrainVertexDeclaration;
 
                 int noVertices = terrainVertexBuffer.SizeInBytes / VertexMultitextured.SizeInBytes;
                 int noTriangles = terrainIndexBuffer.SizeInBytes / sizeof(int) / 3;
                 device.DrawIndexedPrimitives(PrimitiveType.TriangleList, 0, 0, noVertices, 0, noTriangles);
 
                 pass.End();
             }
             effect.End();
         }
 
         private void DrawSkyDome(Matrix currentViewMatrix)
         {
             device.RenderState.DepthBufferWriteEnable = false;
 
             Matrix[] modelTransforms = new Matrix[skyDome.Bones.Count];
             skyDome.CopyAbsoluteBoneTransformsTo(modelTransforms);
 
             Matrix wMatrix = Matrix.CreateTranslation(0, -0.3f, 0) * Matrix.CreateScale(100) * Matrix.CreateTranslation(cameraPosition);
             foreach (ModelMesh mesh in skyDome.Meshes)
             {
                 foreach (Effect currentEffect in mesh.Effects)
                 {
                     Matrix worldMatrix = modelTransforms[mesh.ParentBone.Index] * wMatrix;
                     currentEffect.CurrentTechnique = currentEffect.Techniques["Textured"];
                     currentEffect.Parameters["xWorld"].SetValue(worldMatrix);
                     currentEffect.Parameters["xView"].SetValue(currentViewMatrix);
                     currentEffect.Parameters["xProjection"].SetValue(projectionMatrix);
                     currentEffect.Parameters["xTexture"].SetValue(cloudMap);
                     currentEffect.Parameters["xEnableLighting"].SetValue(false);
                 }
                 mesh.Draw();
             }
             device.RenderState.DepthBufferWriteEnable = true;
         }
 
         private Plane CreatePlane(float height, Vector3 planeNormalDirection, Matrix currentViewMatrix, bool clipSide)
         {
             planeNormalDirection.Normalize();
             Vector4 planeCoeffs = new Vector4(planeNormalDirection, height);
             if (clipSide)
                 planeCoeffs *= -1;
 
             Matrix worldViewProjection = currentViewMatrix * projectionMatrix;
             Matrix inverseWorldViewProjection = Matrix.Invert(worldViewProjection);
             inverseWorldViewProjection = Matrix.Transpose(inverseWorldViewProjection);
 
             planeCoeffs = Vector4.Transform(planeCoeffs, inverseWorldViewProjection);
             Plane finalPlane = new Plane(planeCoeffs);
 
             return finalPlane;
         }
 
         private void DrawRefractionMap()
         {
             Plane refractionPlane = CreatePlane(waterHeight + 1.5f, new Vector3(0,-1,0), viewMatrix, false);
             device.ClipPlanes[0].Plane = refractionPlane;
             device.ClipPlanes[0].IsEnabled = true;
             device.SetRenderTarget(0, refractionRenderTarget);
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
             DrawTerrain(viewMatrix);
             device.ClipPlanes[0].IsEnabled = false;
 
             device.SetRenderTarget(0, null);
             refractionMap = refractionRenderTarget.GetTexture();
         }
     }
 }
```

## Next Steps

[Rendering the Reflection map](Riemers3DXNA4advterrain09reflection)
