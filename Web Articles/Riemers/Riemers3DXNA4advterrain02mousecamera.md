# Adding a first-person mouse camera

Let’s first add a moveable camera to our scene, so later on we can easily move it around our terrain. We’ve already seen a dynamic camera in Series 2 that was based on quaternions. That approach is a little too dynamic for this application, because that camera was able to rotate in really any direction. This was due to the fact that every rotation matrix was based on the previous one, and the mesh could rotate around its forward vector and around its side vector.

> Recipes 2-3 and 2-4 discuss both camera modes in a lot of detail.

Generally, you will want a first-person camera to rotate around the up vector (turning) and around the side vector (looking up-down). So instead of storing a rotation matrix, we’ll simply be storing the rotation values around the side and up vector, next to the position of the camera. So we end up with these variables to store our camera definition, which we should put at the top of our code:

```csharp
 Vector3 cameraPosition = new Vector3(130, 30, -50);
 float leftrightRot = MathHelper.PiOver2;
 float updownRot = -MathHelper.Pi / 10.0f;
 const float rotationSpeed = 0.3f;
 const float moveSpeed = 30.0f;
```

The last 2 values will indicate how fast we want our camera to respond on mouse and keyboard input, and should therefore remain constant.

We’ll start by creating the UpdateViewMatrix method, which creates a view matrix based on the current camera position and rotation values. As explained in Series 1 and in more detail in Recipe 2-1, to create the view matrix, we need the camera position, a point where the camera looks at, and the vector that’s considered to be ‘up’ by the camera. These 2 last vectors depend on the rotation of the camera, so we first need to construct the camera rotation matrix based on the current rotation values:

```csharp
 private void UpdateViewMatrix()
 {
     Matrix cameraRotation = Matrix.CreateRotationX(updownRot) * Matrix.CreateRotationY(leftrightRot);

     Vector3 cameraOriginalTarget = new Vector3(0, 0, -1);
     Vector3 cameraRotatedTarget = Vector3.Transform(cameraOriginalTarget, cameraRotation);
     Vector3 cameraFinalTarget = cameraPosition + cameraRotatedTarget;

     Vector3 cameraOriginalUpVector = new Vector3(0, 1, 0);
     Vector3 cameraRotatedUpVector = Vector3.Transform(cameraOriginalUpVector, cameraRotation);

     viewMatrix = Matrix.CreateLookAt(cameraPosition, cameraFinalTarget, cameraRotatedUpVector);
 }
```

The first line creates the camera rotation matrix, by combining the rotation around the X axis (looking up-down) by the rotation around the Y axis (looking left-right).

As target point for our camera, we take the position of our camera, plus the (0,0,-1) ‘forward’ vector. We need to transform this forward vector with the rotation of the camera, so it becomes the forward vector of the camera. We find the ‘Up’ vector the same way: by transforming it with the camera rotation.

With these vectors known, it’s easy to construct the viewMatrix.

Instead of setting the View matrix fixed in the LoadContent method, replace that line with a call to the UpdateViewMatrix method:

```csharp
 UpdateViewMatrix();
```

Next, we’ll make our camera react to user input. We want our camera to go forward/backward or strafe left/right when we press the corresponding arrow buttons on the keyboard. Camera rotation will be directed by mouse input. We’ll create a method for this, that takes the amount of time that has passes since the last call, so the camera will turn at a speed not dependant of the computer speed:

```csharp
 private void ProcessInput(float amount)
 {
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
```

This method will read out the keyboard as explained in Series 2, and fill a moveVector accordingly. This moveVector is multiplied by the amount variable, which will indicate the amount of time passed since the last call. The result is passed to the AddToCameraPosition method, which is described below.

```csharp
 private void AddToCameraPosition(Vector3 vectorToAdd)
 {
     Matrix cameraRotation = Matrix.CreateRotationX(updownRot) * Matrix.CreateRotationY(leftrightRot);
     Vector3 rotatedVector = Vector3.Transform(vectorToAdd, cameraRotation);
     cameraPosition += moveSpeed * rotatedVector;
     UpdateViewMatrix();
 }
```

This method again creates the rotation matrix of the camera. Once you have this matrix, you use it transform the moveDirection. This is needed, because if you want the camera to move into the Forward direction, you don’t want it to move into the (0,0,-1) direction, but into the direction that is actually Forward for the camera.

The last line transforms this vector by the rotation of our camera, so ‘Forward’ will actually be ‘Forward’ relative to our camera.

Of course we still need to call the ProcessInput method from within the Update method:

```csharp
 float timeDifference = (float)gameTime.ElapsedGameTime.TotalMilliseconds / 1000.0f;
 ProcessInput(timeDifference);
```

Running this code, you should be able to move the camera using the arrow buttons.

So far for the movement, let’s see how we can trigger the rotation by our mouse. We can read out the mouse just like we did with the keyboard: by reading the MouseState. This MouseState structure contains info like the absolute X and Y position of the mouse cursor, but not the amount of movement since the last call.

So we need this workaround. At the end of every frame, we’ll reposition the mouse cursor to the middle of the screen. By comparing the current position of the mouse to the middle position of the screen, we can check every frame how much the mouse has moved!

So we’ll have to add this variable:

```csharp
 MouseState originalMouseState;
```

During the startup of your project, you’ll want to position the mouse in the middle and store this state, by putting this code in your LoadContent method:

```csharp
 Mouse.SetPosition(device.Viewport.Width / 2, device.Viewport.Height / 2);
 originalMouseState = Mouse.GetState();
```

And this is the code we need at the top of our ProcessInput method:

```csharp
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
```

First, we retrieve the current MouseState. Next, we check whether this state differs from the original mouse position, in the middle of the screen. If there is a difference, the corresponding rotation values are updated according to the amount of mouse movement, multiplied by the time elapsed since the last frame.

That’s it! When you run the code, you should be able to walk over your terrain as in any other first person game.

> **Attention:** as we’re continuously setting the mouse cursor to the middle of our window, there will be now way to click on the X to close the window. Use Alt+F4 to close the running program

That’s it for this first chapter. Although it doesn’t change a lot to the screenshot, I preferred to start with the camera chapter as it’ll come in useful in later chapters.

![Camera](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-02Camera1.jpg?raw=true)

BTW.. I’ve used the phrase ‘since the last frame’ quite a few times already. Note that this means ‘since the last call of the Update method’, and not ‘since the last time our scene was drawn’. Both actions have been separated in XNA, and the frequency of execution of the Draw method will be adjusted so the Update method will always be called 60 times per second.

## Code so far

As there are no changes to the HLSL code yet, I only have to list the XNA code:

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
         }
 
         private void LoadVertices()
         {

            Texture2D heightMap = Content.Load<Texture2D> ("heightmap");            LoadHeightData(heightMap);

            VertexPositionNormalColored[] terrainVertices = SetUpTerrainVertices();
            int[] terrainIndices = SetUpTerrainIndices();
            terrainVertices = CalculateNormals(terrainVertices, terrainIndices);
            CopyToTerrainBuffers(terrainVertices, terrainIndices);
            terrainVertexDeclaration = new VertexDeclaration(device, VertexPositionNormalColored.VertexElements);
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

        private VertexPositionNormalColored[] SetUpTerrainVertices()
        {
            VertexPositionNormalColored[] terrainVertices = new VertexPositionNormalColored[terrainWidth * terrainLength];

            for (int x = 0; x < terrainWidth; x++)
            {
                for (int y = 0; y < terrainLength; y++)
                {
                    terrainVertices[x + y * terrainWidth].Position = new Vector3(x, heightData[x, y], -y);

                    if (heightData[x, y] < 6)
                        terrainVertices[x + y * terrainWidth].Color = Color.Blue;
                    else if (heightData[x, y] < 15)
                        terrainVertices[x + y * terrainWidth].Color = Color.Green;
                    else if (heightData[x, y] < 25)
                        terrainVertices[x + y * terrainWidth].Color = Color.Brown;
                    else
                        terrainVertices[x + y * terrainWidth].Color = Color.White;
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

        private VertexPositionNormalColored[] CalculateNormals(VertexPositionNormalColored[] vertices, int[] indices)
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

        private void CopyToTerrainBuffers(VertexPositionNormalColored[] vertices, int[] indices)
        {
            terrainVertexBuffer = new VertexBuffer(device, vertices.Length * VertexPositionNormalColored.SizeInBytes, BufferUsage.WriteOnly);
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
             effect.CurrentTechnique = effect.Techniques["Colored"];
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
 
                 device.Vertices[0].SetSource(terrainVertexBuffer, 0, VertexPositionNormalColored.SizeInBytes);
                 device.Indices = terrainIndexBuffer;
                 device.VertexDeclaration = terrainVertexDeclaration;
 
                 int noVertices = terrainVertexBuffer.SizeInBytes / VertexPositionNormalColored.SizeInBytes;
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

[Texturing our terrain](Riemers3DXNA4advterrain03texturedterrain)
