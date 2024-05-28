# Multitexturing

Our textured terrain already looks a lot nicer than a simply colored terrain, but it could still be improved a lot. For example, by attaching different textures according to the height level of each vertex. In this chapter, we’ll divide our terrain into sand, grass, rocky and snow-covered areas. For this, we’ll need the three other textures as well, which you can download here: sand, rocks and snow.

Once again, import them into our project, and declare these variables:

```csharp
 Texture2D sandTexture;
 Texture2D rockTexture;
 Texture2D snowTexture;
```

And fill them in the small LoadTextures method:

```csharp
sandTexture = Content.Load<Texture2D> ("sand");
rockTexture = Content.Load<Texture2D> ("rock");
snowTexture = Content.Load<Texture2D> ("snow");
```

We could do this the same way as we did with the colors, but then we would have the same problem: we would clearly see the edges between 2 different texures. What we want is a smooth transition between the 4 textures. This is obtained by giving each vertex a combination of multiple textures.

Each vertex will store 4 weights, one for each texture. For example: the highest vertex of the terrain would have weight 1 for the snow texture, and height 0 for the other 3 textures, so that vertex would get its color entirely from the snow texture. A vertex in the middle between the snowy and the rocky region will have weight 0.5 for both the snow and rock texture and weight 0 for the other 2 textures, which would result in a color taken for 50% fromt the snow texture and 50% from the rock texture.

So first, we’ll have to define a new custom vertex format, which enables us to store these 4 weights for every vertex, next to the position, normal and texture coordinates. We’ll call it VertexMultitextured:

```csharp
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
```

The last entry is new here: for each vertex we’ll be storing an additional vector4, which will contain the 4 weights. Remember we need to pass a semantic with each entry? This describes to which variable in our vertex that entry will be connected to. Because there’s no standard semantic called textureweights or something, we’ll pass it also as an additional TextureCoordinate. Because we’re already passing another TextureCoordinate, we have to give this the index 1, instead of 0. This is the last argument of each entry, and enables us to pass in more than one of each semantic.

For a detailed explanation on defining your custom vertex format, see Recipe 5-14).

Now we have a new vertex type, replace all instances in your code of VertexPositionNormalTexture by VertexMultitextured using Ctrl+H.

Now each vertex has an additional Vector4 to store the weights, let’s fill them. For this, we need a mapping scheme that maps the height of each vertex into texture weights. To make things easy, you usually want weights to have a value between 0 and 1, where 0 means ‘nothing’ and 1 means ‘full’. This means in each vertex, we need to find 4 weight between 0 and 1. Have a look at the left image below, which shows the weight we ideally want to find. Note that the horizontal X axis indicates the weight values.

![Weight Mapping](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-04Multitexture1.jpg?raw=true)

In the LoadHeightData, the heights of our vertices are set to values between 0 and 30. These heights are represented by the vertical arrow. You can see the vertices with height 0 should have weight=1 for the sand texture, while the vertices with height 30 should have weight=1 for the snow texture. A vertex with height=25 should get a weight between 0 and 1 for both the snow and the rock texture.

This mapping is done by the last 4 lines of the following code, which you should put into our SetUpTerrainVertices method:

```csharp
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
     }
 }
```

Only the last 4 lines are new; they perform the mapping from height to texture weights as shown in part A of the image above. As an explanation, we’ll discuss how to find the weight for the grass texture, TexWeights.Y. You see in the left image that a vertex with height 12 should have weight 1. The weight should become 0 for heights 6 and 18, which are 12-6 and 12+6. In other words: all heights that are within 6 meters from 12 meter high should have a weight factor for the grass texture. This explains the ‘abs(height-12)/6’: it will be 0 for height = 12, and become 1 as height approaches 6 or 18. But we need the opposite: at height 12 we need weight=1, and at heights 6 and 12 we need weight 0. So we subtract our line above from 1 and get ‘1- abs(height-12)/6’. This will become smaller than 0 for height lower than 6 and larger than 18, so we clamp this value between 0 and 1.

Although this is a step in the good direction, it isn’t perfect yet. For example: as their snow and rock weights are 0.2, the pixels corresponding to height 25 will get 20% of their color from the snow texture, and 20% from the rock texture. The remaining 60% will remain black, so they will look very dark. To solve this, we must make sure that for every vertex, the sum of all weights is exactly 1.

To do this, for each vertex we’ll make the sum of all weights, and divide all weights by this sum. In case of the previous example, the sum would be 0.2 + 0.2 = 0.4. Next, 0.2 divided by 0.4 gives 0.5 for both the new snow and rock weights. And of course, 0.5 + 0.5 equals 1. This is what is shown in the right part of the image above. You’ll notice that for each height, the summed weight value is 1.

This normalization is done by the next code, which must be performed for each vertex, so it must be placed inside the double for loop:

```csharp
 float total = terrainVertices[x + y * terrainWidth].TexWeights.X;
 total += terrainVertices[x + y * terrainWidth].TexWeights.Y;
 total += terrainVertices[x + y * terrainWidth].TexWeights.Z;
 total += terrainVertices[x + y * terrainWidth].TexWeights.W;
 
 terrainVertices[x + y * terrainWidth].TexWeights.X /= total;
 terrainVertices[x + y * terrainWidth].TexWeights.Y /= total;
 terrainVertices[x + y * terrainWidth].TexWeights.Z /= total;
 terrainVertices[x + y * terrainWidth].TexWeights.W /= total;
```

Now we have texture weights for each vertex of our terrain and a vertex structure that allows our XNA program to pass these values to the HLSL effect, it’s time to actually code this effect. So open up Series4Effects.fx, where we’ll add a new technique: MultiTextured.

Feel free to delete the existing Colored technique of the file, as we’re not going to use it anymore in this series.

Make sure these XNA-to-HLSL variables are still present at the top of the file:

```csharp
//------- Constants --------
float4x4 xView;
float4x4 xProjection;
float4x4 xWorld;
float3 xLightDirection;
float xAmbient;
bool xEnableLighting;
```

Next, add the necessary texture samplers:

```csharp
//------- Texture Samplers --------
Texture xTexture0;

sampler TextureSampler0 = sampler_state { texture = <xTexture0> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture1;

sampler TextureSampler1 = sampler_state { texture = <xTexture1> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture2;

sampler TextureSampler2 = sampler_state { texture = <xTexture2> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture3;

sampler TextureSampler3 = sampler_state { texture = <xTexture3> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
```

Next, the output structure of the vertex shader that will be used in our Multitextured technique:

```csharp
struct MTVertexToPixel
{
    float4 Position         : POSITION;
    float4 Color            : COLOR0;
    float3 Normal            : TEXCOORD0;
    float2 TextureCoords    : TEXCOORD1;
    float4 LightDirection    : TEXCOORD2;
    float4 TextureWeights    : TEXCOORD3;
};
```

Note this is exactly the same structure as for the Textured technique of Series3, only the TextureWeights entry has been added.

Now it’s time for our vertex shader, which is entirely the same as for the Textured technique, except for the texture weights which I’ll explain below:

```csharp
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

    return Output;
}
```

Note how the data enters our vertex shader: the usual texture coordinates are passed as TEXCOORD0, while our newly added texture weights are using the TEXCOORD1 semantic. Check our vertex definition structure to remember why. This data is simply passed on to the output.
For a more detailed explanation of the Vertex shader, go through Series 3.

Next, the pixel shader. This one only needs to calculate the color of each pixel, so we’ll use the default output structure:

```csharp
struct MTPixelToFrame
{
    float4 Color : COLOR0;
};
```

And now the most interesting code of this chapter: the pixel shader. Start with this code, that handles default lighting stuff:

```csharp
MTPixelToFrame MultiTexturedPS(MTVertexToPixel PSIn)
{
    MTPixelToFrame Output = (MTPixelToFrame)0;

    float lightingFactor = 1;
    if (xEnableLighting)
        lightingFactor = saturate(saturate(dot(PSIn.Normal, PSIn.LightDirection)) + xAmbient);

    return Output;
}
```

Next, we have to define the color of the pixel. This is actually pretty straightforward: we’ll sample all 4 textures, and multiply these 4 colors with their corresponding weight. The 4 results are all summed together, and we multiply the final result with the lighting factor:

```csharp
Output.Color = tex2D(TextureSampler0, PSIn.TextureCoords)*PSIn.TextureWeights.x;
Output.Color += tex2D(TextureSampler1, PSIn.TextureCoords)*PSIn.TextureWeights.y;
Output.Color += tex2D(TextureSampler2, PSIn.TextureCoords)*PSIn.TextureWeights.z;
Output.Color += tex2D(TextureSampler3, PSIn.TextureCoords)*PSIn.TextureWeights.w;

Output.Color *= lightingFactor;
```

That’s it! Don’t forget to add the technique definition:

```csharp
technique MultiTextured
{
    pass Pass0
    {
        VertexShader = compile vs_1_1 MultiTexturedVS();
        PixelShader = compile ps_2_0 MultiTexturedPS();
    }
}
```

Back in the Draw method of our XNA app, we still have to specify we’ll be using this new technique to draw our terrain, and pass in the 4 textures:

```csharp
 effect.CurrentTechnique = effect.Techniques["MultiTextured"];
 effect.Parameters["xTexture0"].SetValue(sandTexture);
 effect.Parameters["xTexture1"].SetValue(grassTexture);
 effect.Parameters["xTexture2"].SetValue(rockTexture);
 effect.Parameters["xTexture3"].SetValue(snowTexture);
```

When you run this code, you should see a multitextured terrain. Use the camera to have a closer look at the transitions between our textures!

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-04Multitexture2.jpg?raw=true)

Although this multitextured terrain already looks a lot nicer, it still can be improved. Try moving the camera very close to the sand, and notice the pixels in the texture. Next chapter we’ll discover how we can have higher detail closer to the camera.

> You can try these exercises to practice what you've learned:
>
> - Play around with the weight mapping values. For each texture, there are 2 parameters to change: the central height value of the peak, as well as the width of the peak.
> - Open up the heightmap.bmp file in paint, and find the rivers. In the middle of a river, add a few completely white pixels. Run the program, and try to explain what you see.

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
float4x4 xProjection;
float4x4 xWorld;
float3 xLightDirection;
float xAmbient;
bool xEnableLighting;

//------- Texture Samplers --------
Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture0;

sampler TextureSampler0 = sampler_state { texture = <xTexture0> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture1;

sampler TextureSampler1 = sampler_state { texture = <xTexture1> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture2;

sampler TextureSampler2 = sampler_state { texture = <xTexture2> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture3;

sampler TextureSampler3 = sampler_state { texture = <xTexture3> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
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
    
    return Output;    
}

MTPixelToFrame MultiTexturedPS(MTVertexToPixel PSIn)
{
    MTPixelToFrame Output = (MTPixelToFrame)0;        
    
    float lightingFactor = 1;
    if (xEnableLighting)
        lightingFactor = saturate(saturate(dot(PSIn.Normal, PSIn.LightDirection)) + xAmbient);
        
    Output.Color = tex2D(TextureSampler0, PSIn.TextureCoords)*PSIn.TextureWeights.x;
    Output.Color += tex2D(TextureSampler1, PSIn.TextureCoords)*PSIn.TextureWeights.y;
    Output.Color += tex2D(TextureSampler2, PSIn.TextureCoords)*PSIn.TextureWeights.z;
    Output.Color += tex2D(TextureSampler3, PSIn.TextureCoords)*PSIn.TextureWeights.w;    
        
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
```

## Next Steps

[Adding more detail](Riemers3DXNA4advterrain05addingdetail)
