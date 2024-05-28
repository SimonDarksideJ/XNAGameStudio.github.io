# HLSL Cylindrical Billboarding

You can find a fully detailed explanation on Billboarding in Recipe 3-11. This recipe explains spherical and cylindrical billboarding, calculated on the CPU or GPU. Therefore, this chapter will only briefly discuss the theory and focus on the implementation of the code.

With our multitextured terrain and water finished, it’s time to add some trees to our terrain.

We could do this by loading a 3D Model, and rendering it a few hundreds of times on our terrain. The only problem in that case would be that our framerate would drop to less than 1 frame per second.

This is solved using billboarding, a very important and widely used technique in game programming. Billboarding comes down to replacing a distant 3D object by a simple 2D image. So in case of our tree, we would replace a 3D tree Model by a simple 2D tree image, positioned at the same position in our 3D world.

The only thing you have to make sure that the image is rotated so it always faces the camera. The image below shows why:

![Billboarding](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-15Billboarding1.png?raw=true)

The left image shows 5 2D images that are positioned in a 3D world. The right image shows exactly the same 5 2D images, rotated so they face the camera. It’s not hard to imagine that in case the images would contain a tree, the right image would give a nice result.

You will render each billboard using 2 triangles, and thus 6 vertices. For each billboard, the 6 vertices need to contain the correct texture coordinates, and they all simply need to contain the same position: the central bottom position of the billboard, as shown in the image below. That’s easy to specify, as it’s the position where the trunk will hit the terrain.

![Face](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-15Billboarding2.png?raw=true)

Later on we will create a list of position where we want a billboard in our 3D world. The method below accepts this list, and for each position in the list it will generate these 6 vertices. It is taken straight from Recipe 3-11:

```csharp
private void CreateBillboardVerticesFromList(List<Vector3> treeList){
    VertexPositionTexture[] billboardVertices = new VertexPositionTexture[treeList.Count * 6];
    int i = 0;
    foreach (Vector3 currentV3 in treeList)
    {
        billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(0, 0));
        billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(1, 0));
        billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(1, 1));

        billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(0, 0));
        billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(1, 1));
        billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(0, 1));
    }

    treeVertexBuffer = new VertexBuffer(device, billboardVertices.Length * VertexPositionTexture.SizeInBytes, BufferUsage.WriteOnly);
    treeVertexBuffer.SetData(billboardVertices);
    treeVertexDeclaration = new VertexDeclaration(device, VertexPositionTexture.VertexElements);
}
```

The whole array of vertices os stored in a VertexBuffer, which we need to add as variable to the top of our code:

```csharp
 VertexBuffer treeVertexBuffer;
 VertexDeclaration treeVertexDeclaration;
```

These 6 vertices will be transformed to the correct location by our vertex shader, so that the resulting billboards will always be facing the camera.

The HLSL code that performs this whole calculation is taken straight from Recipe 3-11 and can be downloaded by clicking on this link.

The Recipe assumes the position specified in the 6 vertices is in the middle of the billboard. Since in our case the position we specified is the middle-bottom position, I changed one line in the vertex shader:

```csharp
finalPosition += (1.5f-inTexCoord.y*1.5f)*upVector;
```

This line has already been changed in the bbEffect.fx file you downloaded.

Make sure you import the bbEffect.fx file into the Content entry of your Project, and add the variable:

```csharp
 Effect bbEffect;
```

And load it in the LoadContent method:

```csharp
bbEffect = Content.Load<Effect> ("bbEffect");
```

Let’s create a simple method that generates a list with a few positions where we want to put some trees:

```csharp
private List<Vector3> GenerateTreePositions(VertexMultitextured[] terrainVertices){

    List<Vector3> treeList = new List<Vector3> ();
    treeList.Add(terrainVertices[3310].Position);
    treeList.Add(terrainVertices[3315].Position);
    treeList.Add(terrainVertices[3320].Position);
    treeList.Add(terrainVertices[3325].Position);

    return treeList;
}
```

This method creates a list, carrying 4 positions I chose quite randomly on our terrain. The next chapter we’re going to extend this method, for now we’ll try to render these 4 trees to the screen.

Make sure you call both methods from within our LoadVertices method:

```csharp
List<Vector3> treeList = GenerateTreePositions(terrainVertices);CreateBillboardVerticesFromList(treeList);
```

We still need to import a 2D image of a tree, which you can download by clicking here. Import the image into your Project, and add this variable:

```csharp
 Texture2D treeTexture;
```

Load it in our LoadTextures method:

```csharp
treeTexture = Content.Load<Texture2D> ("tree");
```
Right now we have the effect, texture and vertices ready, so we’re ready to render the triangles. This is done using this method:

```csharp
 private void DrawBillboards(Matrix currentViewMatrix)
 {
     bbEffect.CurrentTechnique = bbEffect.Techniques["CylBillboard"];
     bbEffect.Parameters["xWorld"].SetValue(Matrix.Identity);
     bbEffect.Parameters["xView"].SetValue(currentViewMatrix);
     bbEffect.Parameters["xProjection"].SetValue(projectionMatrix);
     bbEffect.Parameters["xCamPos"].SetValue(cameraPosition);
     bbEffect.Parameters["xAllowedRotDir"].SetValue(new Vector3(0, 1, 0));
     bbEffect.Parameters["xBillboardTexture"].SetValue(treeTexture);

     bbEffect.Begin();
     foreach (EffectPass pass in bbEffect.CurrentTechnique.Passes)
     {
         pass.Begin();
         device.Vertices[0].SetSource(treeVertexBuffer, 0, VertexPositionTexture.SizeInBytes);
         device.VertexDeclaration = treeVertexDeclaration;
         int noVertices = treeVertexBuffer.SizeInBytes / VertexPositionTexture.SizeInBytes;
         int noTriangles = noVertices / 3;
         device.DrawPrimitives(PrimitiveType.TriangleList, 0, noTriangles);
         pass.End();
     }
     bbEffect.End();
 }
```

We select the cylindrical billboarding technique from the .fx file, and set the necessary XNA-to-HLSL variables. The most important one is the xAllowedRotDir variable, which specifies the axis around which the 2D images are allowed to rotate. This is the difference between spherical and cylindrical billboarding: in spherical billboarding, the 2D image is allowed to rotate along all axis to rotate the image to the camera. As here we’re dealing with trees, we want the tree only to be rotated along its trunk, which is around the (0,1,0) Up axis.

Now call this method at the very end of our Draw method:

```csharp
 DrawBillboards(viewMatrix);
```

Now when you run this code, you should see the 4 images in the left corner of your terrain! Try to move your camera around them, your graphics card will render them so they are always facing the camera!

But you can still see they are rectangular images, because of the black border around the tree. Luckily, our tree texture contains transparency information, so enable alpha blending by putting this code in the DrawBillboards method, before you actually render the billboards:

```csharp
 device.RenderState.AlphaBlendEnable = true;
 device.RenderState.SourceBlend = Blend.SourceAlpha;
 device.RenderState.DestinationBlend = Blend.InverseSourceAlpha;
```

Which will enable regular alpha blending:

![Formula](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-15Billboarding3.png?raw=true)

Don’t forget to disable alpha blending at the end of the method, or the next frame your whole 3D world will be blended!

```csharp
 device.RenderState.AlphaBlendEnable = false;
```

Running this code, you should get the screen below:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-15Billboarding4.jpg?raw=true)

This chapter described the basic of billboarding. Billboarding shows its full potential when you’re rendering hundreds of billboards. We will do this in the next chapter, where we’re going to add a few forests to our world.

## Code so far

The XNA code for this chapter:

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
 
         VertexBuffer treeVertexBuffer;
         VertexDeclaration treeVertexDeclaration;
 
         Effect effect;
         Effect bbEffect;
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
         Texture2D treeTexture;
 
         Model skyDome;
 
         const float waterHeight = 5.0f;
         RenderTarget2D refractionRenderTarget;
         Texture2D refractionMap;
         RenderTarget2D reflectionRenderTarget;
         Texture2D reflectionMap;
 
         Vector3 windDirection = new Vector3(0, 0, 1);
 
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
            bbEffect = Content.Load<Effect> ("bbEffect");            UpdateViewMatrix();
            projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 0.3f, 1000.0f);

            Mouse.SetPosition(device.Viewport.Width / 2, device.Viewport.Height / 2);
            originalMouseState = Mouse.GetState();


            skyDome = Content.Load<Model> ("dome"); skyDome.Meshes[0].MeshParts[0].Effect = effect.Clone(device);
            PresentationParameters pp = device.PresentationParameters;
            refractionRenderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, 1, device.DisplayMode.Format);
            reflectionRenderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, 1, device.DisplayMode.Format);

            LoadVertices();
            LoadTextures();
        }

        private void LoadVertices()
        {

            Texture2D heightMap = Content.Load<Texture2D> ("heightmap"); LoadHeightData(heightMap);
            VertexMultitextured[] terrainVertices = SetUpTerrainVertices();
            int[] terrainIndices = SetUpTerrainIndices();
            terrainVertices = CalculateNormals(terrainVertices, terrainIndices);
            CopyToTerrainBuffers(terrainVertices, terrainIndices);
            terrainVertexDeclaration = new VertexDeclaration(device, VertexMultitextured.VertexElements);

            SetUpWaterVertices();
            waterVertexDeclaration = new VertexDeclaration(device, VertexPositionTexture.VertexElements);



            List<Vector3> treeList = GenerateTreePositions(terrainVertices);            CreateBillboardVerticesFromList(treeList);

         }
 
         private void LoadTextures()
         {

            grassTexture = Content.Load<Texture2D> ("grass");
            sandTexture = Content.Load<Texture2D> ("sand");
            rockTexture = Content.Load<Texture2D> ("rock");
            snowTexture = Content.Load<Texture2D> ("snow");
            cloudMap = Content.Load<Texture2D> ("cloudMap");
            waterBumpMap = Content.Load<Texture2D> ("waterbump");

            treeTexture = Content.Load<Texture2D> ("tree");
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
 

        private void CreateBillboardVerticesFromList(List<Vector3> treeList)        {
            VertexPositionTexture[] billboardVertices = new VertexPositionTexture[treeList.Count * 6];
            int i = 0;
            foreach (Vector3 currentV3 in treeList)
            {
                billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(0, 0));
                billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(1, 0));
                billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(1, 1));

                billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(0, 0));
                billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(1, 1));
                billboardVertices[i++] = new VertexPositionTexture(currentV3, new Vector2(0, 1));
            }

            treeVertexBuffer = new VertexBuffer(device, billboardVertices.Length * VertexPositionTexture.SizeInBytes, BufferUsage.WriteOnly);
            treeVertexBuffer.SetData(billboardVertices);
            treeVertexDeclaration = new VertexDeclaration(device, VertexPositionTexture.VertexElements);
        }


        private List<Vector3> GenerateTreePositions(VertexMultitextured[] terrainVertices)        {

            List<Vector3> treeList = new List<Vector3> ();
            treeList.Add(terrainVertices[3310].Position);
            treeList.Add(terrainVertices[3315].Position);
            treeList.Add(terrainVertices[3320].Position);
            treeList.Add(terrainVertices[3325].Position);

            return treeList;
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
 
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.White, 1.0f, 0);
             DrawSkyDome(viewMatrix);
             DrawTerrain(viewMatrix);
             DrawWater(time);
             DrawBillboards(viewMatrix);
 
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
             Plane refractionPlane = CreatePlane(waterHeight + 1.5f, new Vector3(0, -1, 0), viewMatrix, false);
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
             Plane reflectionPlane = CreatePlane(waterHeight - 0.5f, new Vector3(0, -1, 0), reflectionViewMatrix, true);
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
             effect.Parameters["xCamPos"].SetValue(cameraPosition);
             effect.Parameters["xTime"].SetValue(time);
             effect.Parameters["xWindForce"].SetValue(0.002f);
             effect.Parameters["xWindDirection"].SetValue(windDirection);
 
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
 
         private void DrawBillboards(Matrix currentViewMatrix)
         {
             bbEffect.CurrentTechnique = bbEffect.Techniques["CylBillboard"];
             bbEffect.Parameters["xWorld"].SetValue(Matrix.Identity);
             bbEffect.Parameters["xView"].SetValue(currentViewMatrix);
             bbEffect.Parameters["xProjection"].SetValue(projectionMatrix);
             bbEffect.Parameters["xCamPos"].SetValue(cameraPosition);
             bbEffect.Parameters["xAllowedRotDir"].SetValue(new Vector3(0, 1, 0));
             bbEffect.Parameters["xBillboardTexture"].SetValue(treeTexture);
             
             device.RenderState.AlphaBlendEnable = true;
             device.RenderState.SourceBlend = Blend.SourceAlpha;
             device.RenderState.DestinationBlend = Blend.InverseSourceAlpha;
 
             bbEffect.Begin();
             foreach (EffectPass pass in bbEffect.CurrentTechnique.Passes)
             {
                 pass.Begin();
                 device.Vertices[0].SetSource(treeVertexBuffer, 0, VertexPositionTexture.SizeInBytes);
                 device.VertexDeclaration = treeVertexDeclaration;
                 int noVertices = treeVertexBuffer.SizeInBytes / VertexPositionTexture.SizeInBytes;
                 int noTriangles = noVertices / 3;
                 device.DrawPrimitives(PrimitiveType.TriangleList, 0, noTriangles);
                 pass.End();
             }
             bbEffect.End();
 
             device.RenderState.AlphaBlendEnable = false;
         }
     }
 }

And the contents of the bbEffect.fx file:

//------- XNA interface --------
float4x4 xView;
float4x4 xProjection;
float4x4 xWorld;
float3 xCamPos;
float3 xAllowedRotDir;

//------- Texture Samplers --------
Texture xBillboardTexture;

sampler textureSampler = sampler_state { texture = <xBillboardTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = CLAMP; AddressV = CLAMP;};
struct BBVertexToPixel
{
    float4 Position : POSITION;
    float2 TexCoord    : TEXCOORD0;
};
struct BBPixelToFrame
{
    float4 Color     : COLOR0;
};

//------- Technique: CylBillboard --------
BBVertexToPixel CylBillboardVS(float3 inPos: POSITION0, float2 inTexCoord: TEXCOORD0)
{
    BBVertexToPixel Output = (BBVertexToPixel)0;

    float3 center = mul(inPos, xWorld);
    float3 eyeVector = center - xCamPos;

    float3 upVector = xAllowedRotDir;
    upVector = normalize(upVector);
    float3 sideVector = cross(eyeVector,upVector);
    sideVector = normalize(sideVector);

    float3 finalPosition = center;
    finalPosition += (inTexCoord.x-0.5f)*sideVector;
    finalPosition += (1.5f-inTexCoord.y*1.5f)*upVector;

    float4 finalPosition4 = float4(finalPosition, 1);

    float4x4 preViewProjection = mul (xView, xProjection);
    Output.Position = mul(finalPosition4, preViewProjection);

    Output.TexCoord = inTexCoord;

    return Output;
}

BBPixelToFrame BillboardPS(BBVertexToPixel PSIn) : COLOR0
{
    BBPixelToFrame Output = (BBPixelToFrame)0;
    Output.Color = tex2D(textureSampler, PSIn.TexCoord);

    return Output;
}

technique CylBillboard
{
    pass Pass0
    {        
        VertexShader = compile vs_1_1 CylBillboardVS();
        PixelShader = compile ps_1_1 BillboardPS();        
    }
}
```

## Next Steps

[Region growing](Riemers3DXNA4advterrain16regiongrowing)
