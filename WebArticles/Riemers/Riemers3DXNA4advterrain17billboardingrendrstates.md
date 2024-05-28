# Billboarding – 2 Pass Alpha Blending Renderstates
The trees on our terrain look pretty nice –- at least from some camera positions. If you move your camera to the other side of the terrain, your trees will look like in the image below.

![Trees](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-17Billboards1.jpg?raw=true)

You see that some trees have simply been cut off. So why is this? Let’s see what’s happening on our graphics card:

1. The skydome and terrain are rendered, saving their color in the backbuffer and their distance in the Z buffer.
2. Your program enters the DrawBillboards method, and asks the graphics card to render the billboards.
3. The first billboard is rendered to the screen. Let’s say this is the billboard shown in front in the image above. The whole rectangle of the 2D image is rendered. Also the transparent pixels: the color already present in the backbuffer remains. BUT: since these pixels are actually rendered, they change the value in the Z buffer for these pixels.
4. The second billboard is rendered. Let’s say this is the tree to the right of the front tree. For all pixels of this rectangle, you graphics card will check whether the distance to the camera is closer than the value stored in the Z buffer. This is where the problem is situated: the first tree has already written its distance in the Z buffer, so the pixels of the left part of the second tree have a larger distance to the camera than the value already stored in the Z buffer!

Luckily, there are 2 solutions to this.

The first solution would be to render the trees, starting from back to front. This way, all pixels of all trees will be rendered, allowing perfect blending. The only drawback, however, is that you would need to order the billboards from back to front before rendering them. This order depends entirely on the position of your camera, so you would need to do this reordering each time the camera (or the position of your billboard) moves! Which makes it quite impossible to do for a large number of billboards, as this needs to be done on the CPU.

The second approach is not 100% accurate. The good thing is that it’s 99% accurate, and very fast. This is how we will render our billboards:

1) Start by rendering all billboards, but only the pixels of the trees that are not transparent. This will not render the full rectangle of the images, but only those pixels of the core of tree and leaves. Also, the Z buffer will be updated only for those pixels. You can see the result of this first step in the image below: only the core of the trunks and trees are rendered, but the order of the trees is correct.

![More Trees](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-17Billboards2.jpg?raw=true)

2) In the second step, you will render your billboards again, but this time only the (partly) transparent pixels! While doing this, you will want to disabling updating the Z buffer, because otherwise you will run into the same trouble. You can disable the updating of the Z buffer, because the core of the trees have already been drawn correctly, and thus the most important pixels already have the correct Z value. The result after the second step is shown in the final image below.

Let’s see how we can code this down. Go to our DrawBillboards method, and replace the second part of the code by this:

```csharp
 bbEffect.Begin();
 
 device.Vertices[0].SetSource(treeVertexBuffer, 0, VertexPositionTexture.SizeInBytes);
 device.VertexDeclaration = treeVertexDeclaration;
 int noVertices = treeVertexBuffer.SizeInBytes / VertexPositionTexture.SizeInBytes;
 int noTriangles = noVertices / 3;
 {
     device.RenderState.AlphaTestEnable = true;
     device.RenderState.AlphaFunction = CompareFunction.GreaterEqual;
     device.RenderState.ReferenceAlpha = 200;
 
     bbEffect.CurrentTechnique.Passes[0].Begin();
     device.DrawPrimitives(PrimitiveType.TriangleList, 0, noTriangles);
     bbEffect.CurrentTechnique.Passes[0].End();
 }
 
 device.RenderState.AlphaTestEnable = false;
 bbEffect.End();
```

Because we’ll be rendering everything twice, I reorginazed some stuff. The code that actually renders everything is between the brackets. Before rendering everything, we enable the Alpha Test renderstate, which allows us to kick out the pixels that have an alpha value lower than 200. This will cause only the opaque (=non-transparent) pixels to be rendered.

The second last line is only there so you can already try to render your code at this moment, which should give you the image below. Replace that line with this code, which instructs the graphics card to render the billboards again:

```csharp
 {
     device.RenderState.DepthBufferWriteEnable = false;
 
     device.RenderState.AlphaBlendEnable = true;
     device.RenderState.SourceBlend = Blend.SourceAlpha;
     device.RenderState.DestinationBlend = Blend.InverseSourceAlpha;
 
     device.RenderState.AlphaTestEnable = true;
     device.RenderState.AlphaFunction = CompareFunction.Less;
     device.RenderState.ReferenceAlpha = 200;
 
     bbEffect.CurrentTechnique.Passes[0].Begin();
     device.DrawPrimitives(PrimitiveType.TriangleList, 0, noTriangles);
     bbEffect.CurrentTechnique.Passes[0].End();
 }
 device.RenderState.AlphaBlendEnable = false;
 device.RenderState.DepthBufferWriteEnable = true;
 device.RenderState.AlphaTestEnable = false;
```

The first line disables updating of the Z buffer, while the next block enables alpha blending as we’ve seen in the chapter on billboarding. The next block sets a different Alpha Test rule: this time we only want to render the transparent pixels! The last 3 lines inside the block actually render the billboards.

The 2 lines outside the brackets reset the critical renderstates to their default values, so next frame we’re ready to render our terrain as usual.

Running this code should give you something like this:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-17Billboards3.jpg?raw=true)

And this time, this remains valid no matter where the camera is positioned ;)

That’s it for our billboards! We’ve discussed some theory, seen how to solve some practical transparency issues and how to render them in regions. All of this is absolutely not restricted to trees and forests, but you’ll be able to use it in many cases.

We’ve almost reached the end of the 4th series. Almost, as we still have to discuss one more very important aspect in game programming: noise!

## Code so far

Our XNA code thus far:

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

            bbEffect.Begin();

 
             device.Vertices[0].SetSource(treeVertexBuffer, 0, VertexPositionTexture.SizeInBytes);
             device.VertexDeclaration = treeVertexDeclaration;
             int noVertices = treeVertexBuffer.SizeInBytes / VertexPositionTexture.SizeInBytes;
             int noTriangles = noVertices / 3;
             {
                 device.RenderState.AlphaTestEnable = true;
                 device.RenderState.AlphaFunction = CompareFunction.GreaterEqual;
                 device.RenderState.ReferenceAlpha = 200;
 
                 bbEffect.CurrentTechnique.Passes[0].Begin();
                 device.DrawPrimitives(PrimitiveType.TriangleList, 0, noTriangles);
                 bbEffect.CurrentTechnique.Passes[0].End();
             }
 
             {
                 device.RenderState.DepthBufferWriteEnable = false;
 
                 device.RenderState.AlphaBlendEnable = true;
                 device.RenderState.SourceBlend = Blend.SourceAlpha;
                 device.RenderState.DestinationBlend = Blend.InverseSourceAlpha;
 
                 device.RenderState.AlphaTestEnable = true;
                 device.RenderState.AlphaFunction = CompareFunction.Less;
                 device.RenderState.ReferenceAlpha = 200;
 
                 bbEffect.CurrentTechnique.Passes[0].Begin();
                 device.DrawPrimitives(PrimitiveType.TriangleList, 0, noTriangles);
                 bbEffect.CurrentTechnique.Passes[0].End();
             }
             
             device.RenderState.AlphaBlendEnable = false;
             device.RenderState.DepthBufferWriteEnable = true;
             device.RenderState.AlphaTestEnable = false;            
 
             bbEffect.End();
         }
     }
 }
``` 

## Next Steps

[2D Perlin noise](Riemers3DXNA4advterrain18perlinnoise)
