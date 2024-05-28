# 2D Perlin noise

Noise is a very important aspect in game programming. Noise is not the same as static. Static is a set of totally random datapoints, where each point has nothing to do with its neighboring points. An example of a 2D static map is shown on the left side of the image below.

In a noise map, the value of each point smoothly changes from point to point. So although the value in each point is different, the value of a point is probably close to the values of its neighboring points. An example of a noise map is shown on the right side of the image below.

![Noise Maps](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-18Perlin1.jpg?raw=true)

Thus far in this 4th series, we’ve already been using noise maps. The cloudmap is a nice example, together with the noisemap used for growing our forests. But there’s more you can do with noisemaps. For example, how would you generate a heightmap? That’s right, a heightmap is nothing more than a well-smoothed noisemap. Furthermore, noisemaps can be used to define the borders of countries and such on your terrain.

Now I hope you’re convinced a 2D noise map is quite useful, let’s see how we can generate one. To this end, I’ve coded a HLSL effect that is an adapted version of the Perlin Noise generation. This is how it works. You start from multiple static maps, which are easy to generate as they only contain random numbers. The resolution of the static maps has to be the same, but the level of detail increases by a factor 2 from map to map:

![More Noise Maps](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-18Perlin2.jpg?raw=true)

Now if you all add them together, you get this:

![Combined Noise Map](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-18Perlin3.jpg?raw=true)

Whow that’s easy!

I’m glad you think so. There’s one difficult point in the story: the high resolution image is based on only 5x5 values, but all pixels of the image are linear interpolation between these values. But that’s entirely no problem, as the texture interpolator on our graphics card will do this job automatically for us! Also, instead of using 6 different maps, we’re going to use the same map 6 times, with differently scaled texture coordinates.

So in our XNA code, we will start by coding a small method that generates a low-resolution static map:

```csharp
 private Texture2D CreateStaticMap(int resolution)
 {
     Random rand = new Random();
     Color[] noisyColors = new Color[resolution * resolution];
     for (int x = 0; x < resolution; x++)
     for (int y = 0; y < resolution; y++)
     noisyColors[x + y * resolution] = new Color(new Vector3((float)rand.Next(1000) / 1000.0f, 0, 0));

     Texture2D noiseImage = new Texture2D(device, resolution, resolution, 1, TextureUsage.None, SurfaceFormat.Color);
     noiseImage.SetData(noisyColors);
     return noiseImage;
 }
```

Simply specify the resolution, and this method will return a square texture will completely random values stored as red color component in all its pixels.

Since we are going to render the noise map as a HLSL effect, we’ll need 2 triangles covering the whole screen. 2 triangles need 6 vertices as a TrianleList, or 4 vertices as a TriangleStrip (see Series 3):

```csharp
 private VertexPositionTexture[] SetUpFullscreenVertices()
 {
     VertexPositionTexture[] vertices = new VertexPositionTexture[4];

     vertices[0] = new VertexPositionTexture(new Vector3(-1, 1, 0f), new Vector2(0, 1));
     vertices[1] = new VertexPositionTexture(new Vector3(1, 1, 0f), new Vector2(1, 1));
     vertices[2] = new VertexPositionTexture(new Vector3(-1, -1, 0f), new Vector2(0, 0));
     vertices[3] = new VertexPositionTexture(new Vector3(1, -1, 0f), new Vector2(1, 0));

     return vertices;
 }
```

We will need a separate render target to render this into, so add these variables to the top of our XNA code:

```csharp
 RenderTarget2D cloudsRenderTarget;
 Texture2D cloudStaticMap;
 VertexPositionTexture[] fullScreenVertices;
 VertexDeclaration fullScreenVertexDeclaration;
```

We will use the cloudStaticMap to hold our basic static map. Initiliaze the render target in our LoadContent method:

```csharp
 cloudsRenderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, 1, device.DisplayMode.Format);
```

Generate the static map in the LoadTextures method:

```csharp
 cloudStaticMap = CreateStaticMap(32);
```

Which will generate a 32x32 static map. Quite important, if you want smaller clouds you need to specify a higher number here, like 64.

Finally, load the fullscreen vertices and their VertexDeclaration in the LoadVertices method:

```csharp
 fullScreenVertices = SetUpFullscreenVertices();
 fullScreenVertexDeclaration = new VertexDeclaration(device, VertexPositionTexture.VertexElements);
```

With all of the initialization stuff done, we’re ready to move over to the HLSL code! There’s absolutely nothing fancy about the effect, the vertex shader simply needs to pass the texture coordinates to the pixel shader. So add this code to the end of our Series4Effects.fx file:

```csharp
//------- Technique: PerlinNoise --------
struct PNVertexToPixel
{
    float4 Position         : POSITION;
    float2 TextureCoords    : TEXCOORD0;
};

struct PNPixelToFrame
{
    float4 Color : COLOR0;
};
```

The vertex shader is really the easiest vertex shader you can have. Since we’ve already specified the vertex coordinates in screen space, we simply need to route the input to the output:

```csharp
PNVertexToPixel PerlinVS(float4 inPos : POSITION, float2 inTexCoords: TEXCOORD)
{
    PNVertexToPixel Output = (PNVertexToPixel)0;

    Output.Position = inPos;
    Output.TextureCoords = inTexCoords;

    return Output;
}
```

So let’s hope our pixel shader is a bit more fancy:

```csharp
PNPixelToFrame PerlinPS(PNVertexToPixel PSIn)
{
    PNVertexToPixel Output = (PNVertexToPixel)0;

    float4 perlin = tex2D(TextureSampler, (PSIn.TextureCoords))/2;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*2)/4;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*4)/8;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*8)/16;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*16)/32;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*32)/32;
}
```

This corresponds to the 6 image above: you take the static image, and sample it 6 times at different resolutions! Since we want the perlin value to remain between 0 and 1, we need to scale the result down a bit. The first line, which has the lowest resolution, has the largest influence: 50% of the final result. The next one has 25%, then 12.5% and so on to the highest resolution. Make sure that when you sum everything up, you end with a total of exactly 100%.

If we would let this run and capture the output, we would already have a very nice noise map!! But we can do something way more fancy. Imagine you would use the resulting noise map as cloudmap, and you put it over the skydome. To make it move, you could simply move it over the skybox. Nice, but that’s not what’s going on if you take a look outside. In reality, clouds are not only moving, but they are also changing shape!

As with most things, this can also be programmed into our pixel shader. The trick is to make the images of different resolution scroll over each other at different speeds! This is what you get:

```csharp
PixelToFrame PerlinPS(VertexToPixel PSIn)
{
    PNVertexToPixel Output = (PNVertexToPixel)0;

    float2 move = float2(0,1);
    float4 perlin = tex2D(TextureSampler, (PSIn.TextureCoords)+xTime*move)/2;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*2+xTime*move)/4;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*4+xTime*move)/8;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*8+xTime*move)/16;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*16+xTime*move)/32;
    perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*32+xTime*move)/32;
}
```

According to the current time, the images will be scrolled over each other at different speeds! This will give you a new shifted version for each new xTime value.

Obviously, this pixel shader still misses an output. Add these lines:

```csharp
Output.Color = 1.0f-pow(perlin, xOvercast)*2.0f;
return Output;
```

Since the perlin value is between 0 and 1, you can sharpen up the image by taking it to a power larger than 1. The larger the xOvercast value will be, the smaller and sharper your perlin clouds will be, but since you subtract this value from 1.0f, you get the inverse: the larger the xOvercast value, the more clouds you will have.

Let’s not forget to add this xOvercast XNA-to-HLSL variable to the top of our effect:

```csharp
float xOvercast;
```

And of course the technique definition:

```csharp
technique PerlinNoise
{
    pass Pass0
    {
        VertexShader = compile vs_1_1 PerlinVS();
        PixelShader = compile ps_2_0 PerlinPS();
    }
}
```

That’s it for our HLSL code! Let’s move to our XNA file, where we still need to code a simple method that actually runs this effect:

```csharp
 private void GeneratePerlinNoise(float time)
 {
     device.SetRenderTarget(0, cloudsRenderTarget);
     device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);

     effect.CurrentTechnique = effect.Techniques["PerlinNoise"];
     effect.Parameters["xTexture"].SetValue(cloudStaticMap);
     effect.Parameters["xOvercast"].SetValue(1.1f);
     effect.Parameters["xTime"].SetValue(time/1000.0f);
     effect.Begin();
     foreach (EffectPass pass in effect.CurrentTechnique.Passes)
     {
         pass.Begin();

         device.VertexDeclaration = fullScreenVertexDeclaration;
         device.DrawUserPrimitives(PrimitiveType.TriangleStrip, fullScreenVertices, 0, 2);

         pass.End();
     }
     effect.End();

     device.SetRenderTarget(0, null);
     cloudMap = cloudsRenderTarget.GetTexture();
 }
```

This should be basic stuff by now. You activate the custom render target and clear it to Black. Next, you select the PerlinNoise technique we just coded and pass it the basic static map, together with the xOvercast and xTime values. Then you render the 2 triangles covering the whole render target, and you store the contents into the cloudMap texture, so it’s ready for the next chapter!

Let’s not forget to call this method from the top of our Draw method:

```csharp
 GeneratePerlinNoise(time);
```

If you save the cloudMap texture to file, you would get something like this:

![Cloud Map](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-18Perlin4.jpg?raw=true)

This concludes this chapter on noise map generation. It has become quite lengthy, but I think that’s fine as it is an important topic which I did not manage to cover in my book.

The next and final chapter of this series will use the on-the-fly generated noisemap as texture for the skydome.

## The code so far

Our XNA code:

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
         RenderTarget2D cloudsRenderTarget;
         Texture2D cloudStaticMap;
         VertexPositionTexture[] fullScreenVertices;
         VertexDeclaration fullScreenVertexDeclaration;
 
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

             cloudsRenderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, 1, device.DisplayMode.Format);
 
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

 
             fullScreenVertices = SetUpFullscreenVertices();
             fullScreenVertexDeclaration = new VertexDeclaration(device, VertexPositionTexture.VertexElements);
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
            treeMap = Content.Load<Texture2D> ("treeMap");
             cloudStaticMap = CreateStaticMap(32);
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


         private Texture2D CreateStaticMap(int resolution)
         {
             Random rand = new Random();
             Color[] noisyColors = new Color[resolution * resolution];
             for (int x = 0; x < resolution; x++)
                 for (int y = 0; y < resolution; y++)
                     noisyColors[x + y * resolution] = new Color(new Vector3((float)rand.Next(1000) / 1000.0f, 0, 0));
 
             Texture2D noiseImage = new Texture2D(device, resolution, resolution, 1, TextureUsage.None, SurfaceFormat.Color);
             noiseImage.SetData(noisyColors);
             return noiseImage;
         }
 
         private VertexPositionTexture[] SetUpFullscreenVertices()
         {
             VertexPositionTexture[] vertices = new VertexPositionTexture[4];
 
             vertices[0] = new VertexPositionTexture(new Vector3(-1, 1, 0f), new Vector2(0, 1));
             vertices[1] = new VertexPositionTexture(new Vector3(1, 1, 0f), new Vector2(1, 1));
             vertices[2] = new VertexPositionTexture(new Vector3(-1, -1, 0f), new Vector2(0, 0));
             vertices[3] = new VertexPositionTexture(new Vector3(1, -1, 0f), new Vector2(1, 0));
 
             return vertices;
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
             GeneratePerlinNoise(time);
 
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
 
         private void GeneratePerlinNoise(float time)
         {
             device.SetRenderTarget(0, cloudsRenderTarget);
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
                         
             effect.CurrentTechnique = effect.Techniques["PerlinNoise"];
             effect.Parameters["xTexture"].SetValue(cloudStaticMap);
             effect.Parameters["xOvercast"].SetValue(1.1f);
             effect.Parameters["xTime"].SetValue(time/1000.0f);
             effect.Begin();
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Begin();
 
                 device.VertexDeclaration = fullScreenVertexDeclaration;
                 device.DrawUserPrimitives(PrimitiveType.TriangleStrip, fullScreenVertices, 0, 2);
 
                 pass.End();
             }
             effect.End();
 
             device.SetRenderTarget(0, null);
             cloudMap = cloudsRenderTarget.GetTexture();
         }
     }
 }
```

## Shader code

And our Series4Effect.fx file:

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
float3 xCamPos;
float xTime;
float xWindForce;
float3 xWindDirection;

 float xOvercast;
 
 //------- Texture Samplers --------
 Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture0;

sampler TextureSampler0 = sampler_state { texture = <xTexture0> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture1;

sampler TextureSampler1 = sampler_state { texture = <xTexture1> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture2;

sampler TextureSampler2 = sampler_state { texture = <xTexture2> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture3;

sampler TextureSampler3 = sampler_state { texture = <xTexture3> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xReflectionMap;

sampler ReflectionSampler = sampler_state { texture = <xReflectionMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xRefractionMap;

sampler RefractionSampler = sampler_state { texture = <xRefractionMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xWaterBumpMap;

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
    float4 RefractionMapSamplingPos : TEXCOORD3;
    float4 Position3D                : TEXCOORD4;
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
    Output.RefractionMapSamplingPos = mul(inPos, preWorldViewProjection);
    Output.Position3D = mul(inPos, xWorld);        
    
    float3 windDir = normalize(xWindDirection);    
    float3 perpDir = cross(xWindDirection, float3(0,1,0));
    float ydot = dot(inTex, xWindDirection.xz);
    float xdot = dot(inTex, perpDir.xz);
    float2 moveVector = float2(xdot, ydot);
    moveVector.y += xTime*xWindForce;    
    Output.BumpMapSamplingPos = moveVector/xWaveLength;    

    
    return Output;
}

WPixelToFrame WaterPS(WVertexToPixel PSIn)
{
    WPixelToFrame Output = (WPixelToFrame)0;        
    
    float4 bumpColor = tex2D(WaterBumpMapSampler, PSIn.BumpMapSamplingPos);
    float2 perturbation = xWaveHeight*(bumpColor.rg - 0.5f)*2.0f;
    
    float2 ProjectedTexCoords;
    ProjectedTexCoords.x = PSIn.ReflectionMapSamplingPos.x/PSIn.ReflectionMapSamplingPos.w/2.0f + 0.5f;
    ProjectedTexCoords.y = -PSIn.ReflectionMapSamplingPos.y/PSIn.ReflectionMapSamplingPos.w/2.0f + 0.5f;        
    float2 perturbatedTexCoords = ProjectedTexCoords + perturbation;
    float4 reflectiveColor = tex2D(ReflectionSampler, perturbatedTexCoords);
    
    float2 ProjectedRefrTexCoords;
    ProjectedRefrTexCoords.x = PSIn.RefractionMapSamplingPos.x/PSIn.RefractionMapSamplingPos.w/2.0f + 0.5f;
    ProjectedRefrTexCoords.y = -PSIn.RefractionMapSamplingPos.y/PSIn.RefractionMapSamplingPos.w/2.0f + 0.5f;    
    float2 perturbatedRefrTexCoords = ProjectedRefrTexCoords + perturbation;    
    float4 refractiveColor = tex2D(RefractionSampler, perturbatedRefrTexCoords);
    
    float3 eyeVector = normalize(xCamPos - PSIn.Position3D);
    float3 normalVector = (bumpColor.rbg-0.5f)*2.0f;
    
    float fresnelTerm = dot(eyeVector, normalVector);    
    float4 combinedColor = lerp(reflectiveColor, refractiveColor, fresnelTerm);
    
    float4 dullColor = float4(0.3f, 0.3f, 0.5f, 1.0f);
    
    Output.Color = lerp(combinedColor, dullColor, 0.2f);    
    
    float3 reflectionVector = -reflect(xLightDirection, normalVector);
    float specular = dot(normalize(reflectionVector), normalize(eyeVector));
    specular = pow(specular, 256);        
    Output.Color.rgb += specular;
    
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


 //------- Technique: PerlinNoise --------
 struct PNVertexToPixel
 {    
     float4 Position         : POSITION;
     float2 TextureCoords    : TEXCOORD0;
 };
 
 struct PNPixelToFrame
 {
     float4 Color : COLOR0;
 };
 
 PNVertexToPixel PerlinVS(float4 inPos : POSITION, float2 inTexCoords: TEXCOORD)
 {    
     PNVertexToPixel Output = (PNVertexToPixel)0;
     
     Output.Position = inPos;
     Output.TextureCoords = inTexCoords;
     
     return Output;    
 }
 
 PNPixelToFrame PerlinPS(PNVertexToPixel PSIn)
 {
     PNPixelToFrame Output = (PNPixelToFrame)0;    
     
     float2 move = float2(0,1);
     float4 perlin = tex2D(TextureSampler, (PSIn.TextureCoords)+xTime*move)/2;
     perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*2+xTime*move)/4;
     perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*4+xTime*move)/8;
     perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*8+xTime*move)/16;
     perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*16+xTime*move)/32;
     perlin += tex2D(TextureSampler, (PSIn.TextureCoords)*32+xTime*move)/32;    
     
     Output.Color.rgb = 1.0f-pow(perlin.r, xOvercast)*2.0f;
     Output.Color.a =1;
 
     return Output;
 }
 
 technique PerlinNoise
 {
     pass Pass0
     {
         VertexShader = compile vs_1_1 PerlinVS();
         PixelShader = compile ps_2_0 PerlinPS();
     }
 }
```

## Next Steps

[Gradient SkyDome](Riemers3DXNA4advterrain19gradient)
