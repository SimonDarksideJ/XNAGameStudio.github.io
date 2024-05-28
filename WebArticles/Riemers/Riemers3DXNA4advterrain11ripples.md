# Rippling water – Novice bump mapping

Now our water looks like a mirror, we’ll add a rippling effect so it will look a lot more realistic. So exactly how are we going to do this? Instead of really going into sines and cosines (the mathematical equivalent of a wave), we’ll use a very simple trick (simple, once you understand it ;). For each pixel of the water, instead of sampling the reflection map at the correct position, we’ll change the sampling position a little bit differently in each pixel. I’ll call this the texture coordinate perturbation.

One way to do this would be to split the water plane up into lots of small triangles and update their texture coordinates every frame in our XNA app. This would be a HUGE task for the CPU, simply impossible to do. The correct approach would be to stick to our simple 2 triangles, and do all the texture coordinate calculations in the pixel shader.

The question that remains: exactly how far do we have to adjust the texture coordinates of each pixel? To find this out, try to imagine how things look like in nature: if there’s absolutely no wind, the water is flat, and looks like a mirror. If there is wind, you also have waves, but at the top and bottom of the waves, the water is flat and thus there the reflections are still perfect: we need the smallest perturbations at the bottom and top of each wave. However, the flanks of the waves are absolutely not horizontal, hence here we will have the largest perturbations!

What we need is a measurement of the ‘flatness’ of each pixel in the water. For this, we will use an image of real waves. However, we’re not interested in the wave image itself, but in the flatness of each pixel. This is exactly what a gradient map is: if you have a 256x256 wave image A, its gradient map B is also a 256x256 image where the color of each pixel indicates the difference in color of the same pixel in image A.

A gradient map is the mathematical term; in games this usually is called the bump map. Below you can see such a bump map of water, and you can download it here for your project:

![Bump Map](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-11Ripples1.jpg?raw=true)

As always, each pixel of this image contains 3 values, one for each color. These 3 values correspond to the 3 components of the normal vector in the map. For example, if the surface would be completely flat in a certain pixel, the normal vector would be pointing straight upward. So the X and Z components would be 0, while the Y component would be 1. Or in colors, the Red and Green components would be 0, the Blue would be 1, so the whole map would be completely blue (this is not exactly what you’ll find in a bump map, as you’ll learn soon).

However, since the water isn’t flat, the normal is different in each pixel, where the Red and Green components contain exactly how much the normal is pointing in the X and Z directions. So the larger the red and green components, the less the normal vector will point upwards, the less flat the water will be.

The blue color component is only provided so you can create the normal vector, for which you need 3 components, but in this case we’re only interested in how flat/rippled the water is at a certain pixel, so the red and green components will do, as you’ll see later.

Over to the code. In our XNA app, there’s not a lot we need to do: we simply need to pass the bump map to our shader. So declare the variable:

```csharp
 Texture2D waterBumpMap;
```

Fill it in the LoadTextures method:

```csharp
waterBumpMap = Content.Load<Texture2D> ("waterbump");
```

And pass it on to the shader in the DrawWater method:

```csharp
 effect.Parameters["xWaterBumpMap"].SetValue(waterBumpMap);
```

That’s it for the XNA code, let’s move on to the HLSL part. Start by accepting the bump map:

```csharp
Texture xWaterBumpMap;

sampler WaterBumpMapSampler = sampler_state { texture = <xWaterBumpMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
```

We’ll also declare 2 variables we’ll use later on in this chapter:

```csharp
float xWaveLength;
float xWaveHeight;
```

I don’t think their names need further explanation.

Now it’s time to update our shaders. Since the last chapter, we’re already passing texture coordinates with the 6 vertices that make up our water. Now we need them to sample our water bump map, so adjust the vertex output and shader so it will pass them through to the pixel shader:

```csharp
struct WVertexToPixel
{
    float4 Position                 : POSITION;
    float4 ReflectionMapSamplingPos    : TEXCOORD1;
    float2 BumpMapSamplingPos        : TEXCOORD2;
};

Output.BumpMapSamplingPos = inTex/xWaveLength;
```

The last line we divide by the xWaveLength: larger values for this variable will make the tex cords smaller, and thus the bump map will be stretched over a larger area.

On to the pixel shader. There, we first sample the bump map:

```csharp
float4 bumpColor = tex2D(WaterBumpMapSampler, PSIn.BumpMapSamplingPos);
```

At this moment, the red and green color values of the bumpColor indicate how much the sampling coordinates of the reflection map should be perturbated. So you expect it to contain a value between -1 and +1. However, colors can only contain value between 0 and 1! So, as a solution, heightmaps are created by bringing the values from the [-1,1] region into the [0,1] region before saving them as a color. This can be done by dividing by 2 and adding 0.5. As an example, if the normal is unadjusted, you would have the X and Z component would be 0 and the Y component would be 1. When this is shifted to colors, you get (0.5, 0.5, 1), which is the main color you find in most bump maps.

Of course, we need to remap the values we find in the bump map from the [0,1] region into the [-1,1] region, like this:

```csharp
float2 perturbation = xWaveHeight*(bumpColor.rg - 0.5f)*2.0f;
```

And the perturbated texture coordinates finally become:

```csharp
float2 perturbatedTexCoords = ProjectedTexCoords + perturbation;
```

These are the final texture coordinates we can use to sample our reflection map with:

```csharp
Output.Color = tex2D(ReflectionSampler, perturbatedTexCoords);
```

That’s it for the HLSL code, all we need to do is set the 2 variables from within our DrawWater method:

```csharp
 effect.Parameters["xWaveLength"].SetValue(0.1f);
 effect.Parameters["xWaveHeight"].SetValue(0.3f);
```

Running this code should finally give you something that looks like water.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-11Ripples2.jpg?raw=true)

OK, this starts to look like water. Blending the refraction color in will already improve it, just wait until it starts moving!

> You can try these exercises to practice what you've learned:
>
> - Try playing around with the xWaveLength and xWaveHeight values
> - What happens if you put the reflection clipping plane too high or too low?

## Our XNA code thus far

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
 
         VertexBuffer waterVertexBuffer;
         VertexDeclaration waterVertexDeclaration;
 
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
         Texture2D waterBumpMap;
         
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

            SetUpWaterVertices();
            waterVertexDeclaration = new VertexDeclaration(device, VertexPositionTexture.VertexElements);
        }

        private void LoadTextures()
        {

            grassTexture = Content.Load<Texture2D> ("grass");
            sandTexture = Content.Load<Texture2D> ("sand");
            rockTexture = Content.Load<Texture2D> ("rock");
            snowTexture = Content.Load<Texture2D> ("snow");
            cloudMap = Content.Load<Texture2D> ("cloudMap");

            waterBumpMap = Content.Load<Texture2D> ("waterbump");
         }
 
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
 
         private void SetUpWaterVertices()
         {
             VertexPositionTexture[] waterVertices = new VertexPositionTexture[6];
 
             waterVertices[0] = new VertexPositionTexture(new Vector3(0, waterHeight, 0), new Vector2(0, 1));
             waterVertices[2] = new VertexPositionTexture(new Vector3(terrainWidth, waterHeight, -terrainLength), new Vector2(1, 0));
             waterVertices[1] = new VertexPositionTexture(new Vector3(0, waterHeight, -terrainLength), new Vector2(0, 0));
 
             waterVertices[3] = new VertexPositionTexture(new Vector3(0, waterHeight, 0), new Vector2(0, 1));
             waterVertices[5] = new VertexPositionTexture(new Vector3(terrainWidth, waterHeight, 0), new Vector2(1, 1));
             waterVertices[4] = new VertexPositionTexture(new Vector3(terrainWidth, waterHeight, -terrainLength), new Vector2(1, 0));
 
             waterVertexBuffer = new VertexBuffer(device, waterVertices.Length * VertexPositionTexture.SizeInBytes, BufferUsage.WriteOnly);
             waterVertexBuffer.SetData(waterVertices);
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
             DrawWater(time);
 
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
 
         private void DrawWater(float time)
         {
             effect.CurrentTechnique = effect.Techniques["Water"];
             Matrix worldMatrix = Matrix.Identity;
             effect.Parameters["xWorld"].SetValue(worldMatrix);
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xReflectionView"].SetValue(reflectionViewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xReflectionMap"].SetValue(reflectionMap);
             effect.Parameters["xRefractionMap"].SetValue(refractionMap);
             effect.Parameters["xWaterBumpMap"].SetValue(waterBumpMap);
             effect.Parameters["xWaveLength"].SetValue(0.1f);
             effect.Parameters["xWaveHeight"].SetValue(0.3f);
 
             effect.Begin();
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Begin();
 
                 device.Vertices[0].SetSource(waterVertexBuffer, 0, VertexPositionTexture.SizeInBytes);
                 device.VertexDeclaration = waterVertexDeclaration;
                 int noVertices = waterVertexBuffer.SizeInBytes / VertexPositionTexture.SizeInBytes;
                 device.DrawPrimitives(PrimitiveType.TriangleList, 0, noVertices / 3);
 
                 pass.End();
             }
             effect.End();
         }
     }
 }
```

## And our HLSL file

```csharp
//----------------------------------------------------
//--                                                --
//--             www.riemers.net                 --
//--         Series 4: Advanced terrain             --
//--                 Shader code                    --
//--                                                --
//----------------------------------------------------

//------- Constants --------
float4x4 xView;
float4x4 xReflectionView;
float4x4 xProjection;
float4x4 xWorld;
float3 xLightDirection;
float xAmbient;
bool xEnableLighting;

 float xWaveLength;
 float xWaveHeight;


//------- Texture Samplers --------
Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture0;

sampler TextureSampler0 = sampler_state { texture = <xTexture0> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture1;

sampler TextureSampler1 = sampler_state { texture = <xTexture1> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture2;

sampler TextureSampler2 = sampler_state { texture = <xTexture2> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture3;

sampler TextureSampler3 = sampler_state { texture = <xTexture3> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xReflectionMap;

sampler ReflectionSampler = sampler_state { texture = <xReflectionMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xRefractionMap;

sampler RefractionSampler = sampler_state { texture = <xRefractionMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
 Texture xWaterBumpMap;

sampler WaterBumpMapSampler = sampler_state { texture = <xWaterBumpMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};

//------- Technique: Textured --------
struct TVertexToPixel
{
    float4 Position     : POSITION;    
    float4 Color        : COLOR0;
    float LightingFactor: TEXCOORD0;
    float2 TextureCoords: TEXCOORD1;
};

struct TPixelToFrame
{
    float4 Color : COLOR0;
};

TVertexToPixel TexturedVS( float4 inPos : POSITION, float3 inNormal: NORMAL, float2 inTexCoords: TEXCOORD0)
{    
    TVertexToPixel Output = (TVertexToPixel)0;
    float4x4 preViewProjection = mul (xView, xProjection);
    float4x4 preWorldViewProjection = mul (xWorld, preViewProjection);
    
    Output.Position = mul(inPos, preWorldViewProjection);    
    Output.TextureCoords = inTexCoords;
    
    float3 Normal = normalize(mul(normalize(inNormal), xWorld));    
    Output.LightingFactor = 1;
    if (xEnableLighting)
        Output.LightingFactor = saturate(dot(Normal, -xLightDirection));
    
    return Output;    
}

TPixelToFrame TexturedPS(TVertexToPixel PSIn)
{
    TPixelToFrame Output = (TPixelToFrame)0;        
    
    Output.Color = tex2D(TextureSampler, PSIn.TextureCoords);
    Output.Color.rgb *= saturate(PSIn.LightingFactor + xAmbient);

    return Output;
}

technique Textured_2_0
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 TexturedVS();
        PixelShader = compile ps_2_0 TexturedPS();
    }
}

technique Textured
{
    pass Pass0
    {
        VertexShader = compile vs_1_1 TexturedVS();
        PixelShader = compile ps_1_1 TexturedPS();
    }
}

//------- Technique: Multitextured --------
struct MTVertexToPixel
{
    float4 Position         : POSITION;    
    float4 Color            : COLOR0;
    float3 Normal            : TEXCOORD0;
    float2 TextureCoords    : TEXCOORD1;
    float4 LightDirection    : TEXCOORD2;
    float4 TextureWeights    : TEXCOORD3;
    float Depth                : TEXCOORD4;
};

struct MTPixelToFrame
{
    float4 Color : COLOR0;
};

MTVertexToPixel MultiTexturedVS( float4 inPos : POSITION, float3 inNormal: NORMAL, float2 inTexCoords: TEXCOORD0, float4 inTexWeights: TEXCOORD1)
{    
    MTVertexToPixel Output = (MTVertexToPixel)0;
    float4x4 preViewProjection = mul (xView, xProjection);
    float4x4 preWorldViewProjection = mul (xWorld, preViewProjection);
    
    Output.Position = mul(inPos, preWorldViewProjection);
    Output.Normal = mul(normalize(inNormal), xWorld);
    Output.TextureCoords = inTexCoords;
    Output.LightDirection.xyz = -xLightDirection;
    Output.LightDirection.w = 1;    
    Output.TextureWeights = inTexWeights;
    Output.Depth = Output.Position.z/Output.Position.w;
    
    return Output;    
}

MTPixelToFrame MultiTexturedPS(MTVertexToPixel PSIn)
{
    MTPixelToFrame Output = (MTPixelToFrame)0;        
    
    float lightingFactor = 1;
    if (xEnableLighting)
        lightingFactor = saturate(saturate(dot(PSIn.Normal, PSIn.LightDirection)) + xAmbient);
        
    float blendDistance = 0.99f;
    float blendWidth = 0.005f;
    float blendFactor = clamp((PSIn.Depth-blendDistance)/blendWidth, 0, 1);
        
    float4 farColor;
    farColor = tex2D(TextureSampler0, PSIn.TextureCoords)*PSIn.TextureWeights.x;
    farColor += tex2D(TextureSampler1, PSIn.TextureCoords)*PSIn.TextureWeights.y;
    farColor += tex2D(TextureSampler2, PSIn.TextureCoords)*PSIn.TextureWeights.z;
    farColor += tex2D(TextureSampler3, PSIn.TextureCoords)*PSIn.TextureWeights.w;
    
    float4 nearColor;
    float2 nearTextureCoords = PSIn.TextureCoords*3;
    nearColor = tex2D(TextureSampler0, nearTextureCoords)*PSIn.TextureWeights.x;
    nearColor += tex2D(TextureSampler1, nearTextureCoords)*PSIn.TextureWeights.y;
    nearColor += tex2D(TextureSampler2, nearTextureCoords)*PSIn.TextureWeights.z;
    nearColor += tex2D(TextureSampler3, nearTextureCoords)*PSIn.TextureWeights.w;

    Output.Color = lerp(nearColor, farColor, blendFactor);
    Output.Color *= lightingFactor;
    
    return Output;
}

technique MultiTextured
{
    pass Pass0
    {
        VertexShader = compile vs_1_1 MultiTexturedVS();
        PixelShader = compile ps_2_0 MultiTexturedPS();
    }
}

//------- Technique: Water --------
struct WVertexToPixel
{
    float4 Position                 : POSITION;
    float4 ReflectionMapSamplingPos    : TEXCOORD1;

     float2 BumpMapSamplingPos        : TEXCOORD2;
 };
 
 struct WPixelToFrame
 {
     float4 Color : COLOR0;
 };
 
 WVertexToPixel WaterVS(float4 inPos : POSITION, float2 inTex: TEXCOORD)
 {    
     WVertexToPixel Output = (WVertexToPixel)0;
 
     float4x4 preViewProjection = mul (xView, xProjection);
     float4x4 preWorldViewProjection = mul (xWorld, preViewProjection);
     float4x4 preReflectionViewProjection = mul (xReflectionView, xProjection);
     float4x4 preWorldReflectionViewProjection = mul (xWorld, preReflectionViewProjection);
 
     Output.Position = mul(inPos, preWorldViewProjection);
     Output.ReflectionMapSamplingPos = mul(inPos, preWorldReflectionViewProjection);
     Output.BumpMapSamplingPos = inTex/xWaveLength;
 
     return Output;
 }
 
 WPixelToFrame WaterPS(WVertexToPixel PSIn)
 {
     WPixelToFrame Output = (WPixelToFrame)0;        
     
     float2 ProjectedTexCoords;
     ProjectedTexCoords.x = PSIn.ReflectionMapSamplingPos.x/PSIn.ReflectionMapSamplingPos.w/2.0f + 0.5f;
     ProjectedTexCoords.y = -PSIn.ReflectionMapSamplingPos.y/PSIn.ReflectionMapSamplingPos.w/2.0f + 0.5f;    
     
     float4 bumpColor = tex2D(WaterBumpMapSampler, PSIn.BumpMapSamplingPos);
     float2 perturbation = xWaveHeight*(bumpColor.rg - 0.5f)*2.0f;
     float2 perturbatedTexCoords = ProjectedTexCoords + perturbation;
 
     Output.Color = tex2D(ReflectionSampler, perturbatedTexCoords);    
     
     return Output;
 }
 
 technique Water
 {
     pass Pass0
     {
         VertexShader = compile vs_1_1 WaterVS();
         PixelShader = compile ps_2_0 WaterPS();
     }
 }
```

## Next Steps

[Blending in refractions using the Fresnel term](Riemers3DXNA4advterrain12fresnel)
