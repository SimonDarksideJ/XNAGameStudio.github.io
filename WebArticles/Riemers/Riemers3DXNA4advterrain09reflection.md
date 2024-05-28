# Rendering the Reflection map

Last chapter we’ve seen how to set a user clip plane, and how to render the part of the scene that’s under this clip plane to a texture. This chapter, we’ll use the same technique to render the reflection map into its proper texture. This texture is needed, because for each pixel of our water, we need to know the reflective color.

We’ll need another texture to render into, so let’s define these variables:

```csharp
 RenderTarget2D reflectionRenderTarg;
 Texture2D reflectionMap;
```

And initialise them in our LoadContent method:

```csharp
 reflectionRenderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, 1, device.DisplayMode.Format);
```

Now let’s think about how we can render the reflections into a texture. One thing is for sure: we’ll need to reposition our camera for this, as we would like to see the scene as seen by the water. We just need to know where exactly to position our camera, and where it needs to look at.

The exact location our camera should be position, would be the same X and Z coordinates, but the Y coordinate mirrored over the water plane. Have a look at the following image, where this is visualized:

![Water Level](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-09Reflection1.jpg?raw=true)

For each pixel of the water, the reflective color as seen by camera A is the same color as camera B would see. This means that in order to know the reflective colors of each pixel of the water, we have to render the scene as seen by camera B into a texture.

One more problem: as you can see in the image, the part of the terrain under the water obstructs the ray the camera below the water. This means that camera B would only render the bottom of the terrain into the texture! Because of this, we need to clip away all parts of the terrain that are below the water. Luckily, we’ve already seen how to do that in the last chapter.

But first, let’s start by defining the matrix corresponding to camera B, so add this variable to our project:

```csharp
 Matrix reflectionViewMatrix;
```

The UpdateViewMatrix method is where we’ll fill this matrix. As you might know by know, to define a camera matrix, we need a postion, target and up vector for the camera.

We already know the first one: the X and Z coord of camera B is the same as those of camera A, and the Y coord between camera A and the water is the same as the Y coord between the water and camera B. So add this code:

```csharp
 Vector3 reflCameraPosition = cameraPosition;
 reflCameraPosition.Y = -cameraPosition.Y + waterHeight * 2;
```

To obtain the target of camera B, we need to do the same trick:

```csharp
 Vector3 reflTargetPos = cameraFinalTarget;
 reflTargetPos.Y = -cameraFinalTarget.Y + waterHeight * 2;
```

Now we still need to define the Up vector. This is a bit more mathematically, by taking the cross product. The cross product of 2 vectors is the vector that is perpendicular to both vectors. So, if we take the cross product of the forward and side vector of camera B, we get the upvector of camera B. This is the code that does exactly this:

```csharp
 Vector3 cameraRight = Vector3.Transform(new Vector3(1, 0, 0), cameraRotation);
 Vector3 invUpVector = Vector3.Cross(cameraRight, reflTargetPos - reflCameraPosition);
```

Now we have the position, target and up vector for camera B, we’re ready to define its View matrix:

```csharp
 reflectionViewMatrix = Matrix.CreateLookAt(reflCameraPosition, reflTargetPos, invUpVector);
```

With the View matrix of camera B defined, we can create a method DrawReflectionMap where we’ll first define the clip plane. The code below is taken directly from last chapter, where we’ve learned about clip planes:

```csharp
 private void DrawReflectionMap()
 {
    Plane reflectionPlane = CreatePlane(waterHeight - 0.5f, new Vector3(0,-1,0), reflectionViewMatrix, true);
    device.ClipPlanes[0].Plane = reflectionPlane;
    device.ClipPlanes[0].IsEnabled = true;
    device.SetRenderTarget(0, reflectionRenderTarget);
    device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
    DrawTerrain(reflectionViewMatrix);
    DrawSkyDome(reflectionViewMatrix);
    device.ClipPlanes[0].IsEnabled = false;

    device.SetRenderTarget(0, null);
    reflectionMap = reflectionRenderTarget.GetTexture();
 }
```

The first line creates the same plane as in the previous chapter, but the 2 other parameters are different: we need to pass in the view matrix of our B camera, and indicate this time we want to render only the things that are ABOVE the clipping plane.

Next, we activate the clipping plane and render target on our grapics card, and clean the render target. Then the Terrain and Skydome are rendered. Important to note is that this time we pass in the reflectionViewMatrix, as we need the terrain and skybox as seen by camera B!

Don’t forget to call this method from the Draw method:

```csharp
 DrawReflectionMap();
```

That’s it for this chapter. Now, when you add this line:

```csharp
 reflectionMap.Save("refr.jpg", ImageFileFormat.Jpg);
```

You should see this in the ref.jpg file:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-09Reflection2.jpg?raw=true)

The result might look a bit strange at first sight, but it’s not: for each water pixel your camera A looks at, you’ll find the reflective color in this map. So let’s move on to the next chapter, where we’ll actually start using these maps!

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
         Matrix reflectionViewMatrix;
 
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
         RenderTarget2D reflectionRenderTarget;
         Texture2D reflectionMap;
 
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

             reflectionRenderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, 1, device.DisplayMode.Format);
 
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

 
             Vector3 reflCameraPosition = cameraPosition;
             reflCameraPosition.Y = -cameraPosition.Y + waterHeight * 2;
             Vector3 reflTargetPos = cameraFinalTarget;
             reflTargetPos.Y = -cameraFinalTarget.Y + waterHeight * 2;
 
             Vector3 cameraRight = Vector3.Transform(new Vector3(1, 0, 0), cameraRotation);
             Vector3 invUpVector = Vector3.Cross(cameraRight, reflTargetPos - reflCameraPosition);
 
             reflectionViewMatrix = Matrix.CreateLookAt(reflCameraPosition, reflTargetPos, invUpVector);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             float time = (float)gameTime.TotalGameTime.TotalMilliseconds / 100.0f;
 
             DrawRefractionMap();
             DrawReflectionMap();
 
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
 
         private void DrawReflectionMap()
         {
             Plane reflectionPlane = CreatePlane(waterHeight - 0.5f, new Vector3(0,-1,0), reflectionViewMatrix, true);
             device.ClipPlanes[0].Plane = reflectionPlane;
             device.ClipPlanes[0].IsEnabled = true;
             device.SetRenderTarget(0, reflectionRenderTarget);
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
             DrawTerrain(reflectionViewMatrix);
             DrawSkyDome(reflectionViewMatrix);            
             device.ClipPlanes[0].IsEnabled = false;
 
             device.SetRenderTarget(0, null);
             reflectionMap = reflectionRenderTarget.GetTexture();            
         }
     }
 }
```

## Next Steps

[Perfect mirror](Riemers3DXNA4advterrain10perfectmirror)
