# Region growing

This chapter, we’ll be adding a lot more trees to our world. To get the nicest result, instead of simply putting them randomly on our terrain, we’re going to add small patches of forest to our terrain.

The main question that we will tackle in this chapter is: how do create the boundaries of such a forest? The solution is in using a noisemap. In one of the last chapters of this series, you’ll see how you can create noisemaps yourself, for now you can download one by clicking on this link. A smaller version of the image is displayed below:

![Noise Map](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-16Regions1.jpg?raw=true)

The solution is quite simple: for each pixel in the noisemap where the white value is above a certain threshold, we will add a tree to our terrain. So start by importing the treeMap.jpg image into our XNA project.

Next, it’s time to take a look at our GenerateTreePositions method and apply some changes. Our first change will be in the method’s definition, as we’ll need to pass it the noisemap:

```csharp
private List<Vector3> GenerateTreePositions(Texture2D treeMap, VertexMultitextured[] terrainVertices)
You can already change the interior of the method to the code below:

 Color[] treeMapColors = new Color[treeMap.Width * treeMap.Height];
 treeMap.GetData(treeMapColors);
 
 int[,] noiseData = new int[treeMap.Width, treeMap.Height];
 for (int x = 0; x < treeMap.Width; x++)
     for (int y = 0; y < treeMap.Height; y++)
         noiseData[x, y] = treeMapColors[y + x * treeMap.Height].R;
 

List<Vector3> treeList = new List<Vector3> ();
Random random = new Random();

for (int x = 0; x < terrainWidth; x++)
{
    for (int y = 0; y < terrainLength; y++)
    {
        //add trees
    }
}

return treeList;
```

We start by converting the image into a 1D array of colors: first create the 1D array, large enough to store all data and then copy the data from the texture into the array using the GetData method. Next, the 1D array is reshaped into a 2D array of integers. For each pixel, the red component (which is a value between 0 and 255) is taken. Because the current noisemap is a greyscale map, the red component will be the same as the blue and green components.

With our 2D array ready, we create the List of Vector3s, as this will be the final output of the method, and a random number generator as we don’t want our trees to be all lined up.

The final double for loop scans our while terrain grid, and at each vertex we will find out whether we need to add some trees. For each vertex, this is what we need to check:

Is the height acceptable? We don’t want to put some trees in the middle of the river, or on top of our mountains.
Is the terrain flat? We don’t want trees on some steep hills.
What should be the densitity? We want many trees at the center of the forests, while at the borders of the forest the density should be lower.

Let’s tackle these issues one by one.

The first one’s easy: we will simply look up the height of our terrain at the current position, and check if it’s inside an acceptable region. So replace the comment inside the double for-loop by this code:

```csharp
 float terrainHeight = heightData[x, y];
 if ((terrainHeight > 8) && (terrainHeight < 14))
 {
 }
```

This makes sure tree only get planted when the height of the terrain is between 8 and 14.

Next, the steepness. We could add some complex algorithm checking the heights of the neigbouring vertices, but actually we have something much better: the normals inside each vertex. The normal indicates the direction perpendicular to the terrain at each vertex. So on a flat part of the terrain, the normal will point upward.

This is exactly what we’re going to check: how close the normal is to the Up vector. As seen in the Specular highlight chapter, a perfect way to measure this is by taking the dot product of both vectors. So put this line inside the if-check:

```csharp
 float flatness = Vector3.Dot(terrainVertices[x + y * terrainWidth].Normal, new Vector3(0, 1, 0));
```

If the vertex and the (0,1,) Up vector are the same, the dot product will be 1. If they are perpendicular to each other, it will be 0.

Since a dot product is nothing more than the cosine of the angle between both vectors, this is how we can detect the inclination of the terrain is no more than 15 degrees:

```csharp
 float minFlatness = (float)Math.Cos(MathHelper.ToRadians(15));
 if (flatness > minFlatness)
 {
 }
```

15 degrees correspond to a dot product of 0.966, so we want to check whether our terrain is flatter than this.

Finally, we need to check whether the noise map indicates there should be a tree at the current position. Therefore, we will sample the noisemap at the position corresponding to the current location on the terrain. So add this code inside the inner if-check:

```csharp
 float relx = (float)x / (float)terrainWidth;
 float rely = (float)y / (float)terrainLength;
 
 float noiseValueAtCurrentPosition = noiseData[(int)(relx * treeMap.Width), (int)(rely * treeMap.Height)];
 float treeDensity;
 if (noiseValueAtCurrentPosition > 200)
     treeDensity = 5;
 else if (noiseValueAtCurrentPosition > 150)
     treeDensity = 4;
 else if (noiseValueAtCurrentPosition > 100)
     treeDensity = 3;
 else
     treeDensity = 0;
```

Since we want the resolutions of our terrain and noisemap to be independent of each other, we work with relative coordinates. The relx and rely values will be between 0 and 1, allowing us the sample the noisemap at the correct location. The noise value at the current location is stored in the noiseValueAtCurrentPosition variable, which contains a value between 0 and 255.

From this value, we decide how many trees should be added at the current location. If the value is really high, we’re in the middle of a forest and we want to add 5 tree around the current terrain vertex! This number degrades corresponding to the current noise value, and drops to 0 below a certain threshold.

So at this point, we know exactly how many trees we should add at the current terrain vertex! This code actually generates the locations for the new trees, so add it below the code you just entered:

```csharp
 for (int currDetail = 0; currDetail < treeDensity; currDetail++)
 {
     float rand1 = (float)random.Next(1000) / 1000.0f;
     float rand2 = (float)random.Next(1000) / 1000.0f;
 Vector3 treePos = new Vector3((float)x - rand1, 0, -(float)y - rand2); treePos.Y = heightData[x, y];
     treeList.Add(treePos);
 }
```

First you generate 2 random numbers, which you scale to the [0,1] region. You use these to add to the X and Z coordinates of the current vertex, and you use the height of the terrain as Y coordinate. Finally, you add the new position to the List.

Now in our LoadVertices method, we still need to load the treeMap.jpg texture, and pass it to our GenerateTreePositions method! So replace the code at the end of our LoadVertices method with this code:

```csharp
Texture2D treeMap = Content.Load<Texture2D> ("treeMap");
List<Vector3> treeList = GenerateTreePositions(treeMap, terrainVertices);CreateBillboardVerticesFromList(treeList);
```

That’s it! The CreateBillboardVerticesFromList and DrawBillboard methods we defined in the previous chapter will take our work from here, and render 2 triangles for each tree we defined in the GenerateTreePositions method.

Now run this code! You should see a screen very similar to the image below. When you move your camera around the world, there are 2 flaws you might notice:

You’ll notice the most critical flaw when you move the camera above the highest mountain and look at the trees. This error will be the topic of the next chapter.
The trees don’t get reflected in our water! This is because we’re not rendering any trees in our DrawReflectionMap method, so quickly add this call in the method, after the call to the DrawSkyDome method:

```csharp
 DrawBillboards(reflectionViewMatrix);
```

Not at some points you will see the reflections of your trees in the water.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-16Regions2.jpg?raw=true)

We have one major issue to solve on our billboards, which we’ll do in the next chapter.

## Code so far

We’ve only changed our XNA code in this chapter:

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
         Texture2D treeMap;
 
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



            Texture2D treeMap = Content.Load<Texture2D> ("treeMap");
            List<Vector3> treeList = GenerateTreePositions(treeMap, terrainVertices);            CreateBillboardVerticesFromList(treeList);

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
            treeMap = Content.Load<Texture2D> ("treeMap");        }

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



        private List<Vector3> GenerateTreePositions(Texture2D treeMap, VertexMultitextured[] terrainVertices)        {
            Color[] treeMapColors = new Color[treeMap.Width * treeMap.Height];
            treeMap.GetData(treeMapColors);

            int[,] noiseData = new int[treeMap.Width, treeMap.Height];
            for (int x = 0; x < treeMap.Width; x++)
                for (int y = 0; y < treeMap.Height; y++)
                    noiseData[x, y] = treeMapColors[y + x * treeMap.Height].R;


            List<Vector3> treeList = new List<Vector3> ();                        Random random = new Random();

            for (int x = 0; x < terrainWidth; x++)
            {
                for (int y = 0; y < terrainLength; y++)
                {
                    float terrainHeight = heightData[x, y];
                    if ((terrainHeight > 8) && (terrainHeight < 14))
                    {
                        float flatness = Vector3.Dot(terrainVertices[x + y * terrainWidth].Normal, new Vector3(0, 1, 0));
                        float minFlatness = (float)Math.Cos(MathHelper.ToRadians(15));
                        if (flatness > minFlatness)
                        {
                            float relx = (float)x / (float)terrainWidth;
                            float rely = (float)y / (float)terrainLength;

                            float noiseValueAtCurrentPosition = noiseData[(int)(relx * treeMap.Width), (int)(rely * treeMap.Height)];
                            float treeDensity;
                            if (noiseValueAtCurrentPosition > 200)
                                treeDensity = 5;
                            else if (noiseValueAtCurrentPosition > 150)
                                treeDensity = 4;
                            else if (noiseValueAtCurrentPosition > 100)
                                treeDensity = 3;
                            else
                                treeDensity = 0;

                            for (int currDetail = 0; currDetail < treeDensity; currDetail++)
                            {
                                float rand1 = (float)random.Next(1000) / 1000.0f;
                                float rand2 = (float)random.Next(1000) / 1000.0f;
                                Vector3 treePos = new Vector3((float)x - rand1, 0, -(float)y - rand2);
                                treePos.Y = heightData[x, y];
                                treeList.Add(treePos);
                            }
                        }
                    }
                }
            }

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
             DrawSkyDome(reflectionViewMatrix);
             DrawTerrain(reflectionViewMatrix);
             DrawBillboards(reflectionViewMatrix);
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
```

## Next Steps

[Billboarding](Riemers3DXNA4advterrain17billboardingrendrstates)
