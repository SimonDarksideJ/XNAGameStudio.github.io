# Cleaning up our interface using Preshaders

When you take a look at our XNA code where we set the XNA-to-HLSL variables in our effect file, it is a bit disappointing. There is a lot of redundancy in the interface to our HLSL file. First we set the xWorldViewProjection matrix, then the xWorld matrix, the xLightWorldViewProjection matrix, and finally the xViewProjection matrix. You see we pass the world information more than one time.

This way, every new transformation we would like to do in HLSL, would require a new SetValue call, with the data being passed probably already contained in one of the previous sets of data passed to the HLSL code (which is now the case with the world, view and projection data). All of this data has, in fact, nothing to do with the purpose of the method, which has to position and draw the objects in our scene. If we would have to pass a new matrix for each of our following lights, the method would definitely become a mess.

Secondly, for each pixel our last pixel shader is doing a lot of calculations that are the same for all pixels. We would like to do these calculations only once, and use the result for all pixels.

Since we are facing these 2 problems, it’s time we discuss preshaders.

We would like to pass in the view and projection matrices of our camera and light, and the world matrix only once. The question that arises is this: in the end, these matrices have to be multiplied, so where and when would that be?

Up to this point, for each object, the multiplications were done once by our XNA code (in the DrawScene and DrawModel methods), thus done by the CPU (your Intel or AMD processor). This is shown on the left side of the image below: every frame of a game, the game is updated (responding to user input if there is any), the matrices are multiplied and sent to HLSL. Next, the vertex shader is called for every vertex.

![UpdateLoop](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-17Preshaders1.jpg?raw=true)

But we don’t want to do the multiplication in our XNA method, as this litters our code and makes our HLSL interface overly complicated. If we would perform the multiplications in our vertex shader, these multiplications would have to be performed by our GPU (your GeForce or Radeon) in the graphics card for every vertex it has to draw! This is shown in the middle column of the image above.

This is where preshaders kick in: when the HLSL code is being compiled, the compiler checks for code that will be the same for every vertex (or every pixel, in case of the pixel shader). So if we put our matrix multiplications inside our vertex shaders, that part of HLSL code will get stripped away by the compiler, and put in a ‘preshader’. This preshader is executed on the CPU before the vertex shader is actually called, and the resulting constants are passed to the HLSL code. This way, we can code the multiplications in our vertex shader (where they belong, as you’ll see), yet they will be processed only once on the CPU. This is represented on the right part of the image.

Enough for the theory, let’s move on to the code. We’ll put the matrices into pieces: instead of passing the WorldViewProjection matrix, we’ll be passing the World matrix, and the ViewProjection matrix (OK, we could also send the View and Projection matrices separately, but they are always used together).

Let’s start with our camera: this delivers the view and projection matrices. Because we’ll never need them separated, we’ll pass the combined matrix. Replace all lines in our DrawScene method that set effect parameters, and start by adding this one:

```csharp
 effect.Parameters["xCamerasViewProjection"].SetValue(viewMatrix * projectionMatrix);
```

We can do the same for our light

```csharp
 effect.Parameters["xLightsViewProjection"].SetValue(lightsViewProjectionMatrix);
```

Next in line is the World matrix:

```csharp
 effect.Parameters["xWorld"].SetValue(Matrix.Identity);
```

With these 3 matrices, we can make all necessary combinations in our effect file! Of course, we have some remaining variables we need to pass that are not related to the matrices:

```csharp
 effect.Parameters["xTexture"].SetValue(streetTexture);
 effect.Parameters["xLightPos"].SetValue(lightPos);
 effect.Parameters["xLightPower"].SetValue(lightPower);
 effect.Parameters["xAmbient"].SetValue(ambientPower);
 effect.Parameters["xShadowMap"].SetValue(shadowMap);
 effect.Parameters["xCarLightTexture"].SetValue(carLight);
```

To be useful in the DrawModel code, we need to pass parameters to the currentEffect, and we need to pass in the appropriate World matrix and textures:

```csharp
 currentEffect.Parameters["xCamerasViewProjection"].SetValue(viewMatrix * projectionMatrix);
 currentEffect.Parameters["xLightsViewProjection"].SetValue(lightsViewProjectionMatrix);
 currentEffect.Parameters["xWorld"].SetValue(worldMatrix);
 currentEffect.Parameters["xTexture"].SetValue(textures[i++]);
 currentEffect.Parameters["xSolidBrown"].SetValue(solidBrown);
 currentEffect.Parameters["xLightPos"].SetValue(lightPos);
 currentEffect.Parameters["xLightPower"].SetValue(lightPower);
 currentEffect.Parameters["xAmbient"].SetValue(ambientPower);
 currentEffect.Parameters["xShadowMap"].SetValue(shadowMap);
 currentEffect.Parameters["xCarLightTexture"].SetValue(carLight);
```

That’s it for the XNA code. Note that no data is being sent twice. Now it’s time to have a look at the HLSL code, where we have to process the matrix multiplications. First remove all of the old matrix constants in the HLSL code, and replace them with these:

```csharp
float4x4 xCamerasViewProjection;
float4x4 xLightsViewProjection;
float4x4 xWorld;
```

Next in line is our first vertex shader. The SimplestVertexShader (which we don’t use in the last chapters) uses the xWorldViewProjection variable. Now we have to create this constant, starting from the 3 matrices we just defined. Have a look at this code:

float4x4 preWorldViewProjection = mul (xWorld, xCamerasViewProjection);
Output.Position = mul(inPos, preWorldViewProjection);

From the World and the camera’s ViewProjection matrix, we can create the camera’s WorldViewprojection matrix. This WorldViewProjecion matrix will automatically be identified by our compiler as being the same for every vertex of the current object, so it will be extracted by the HLSL compiler to be run on the CPU. This is pre-multipliclation is called the “preshader”. Because of this, I’ve called the result preWorldViewProjection.

With this preWorldViewProjection, each vertex will be multiplied in the vertex shader. The total vertex shader becomes:

```csharp
VertexToPixel SimplestVertexShader( float4 inPos : POSITION0, float3 inNormal: NORMAL0, float2 inTexCoords : TEXCOORD0)
{
    VertexToPixel Output = (VertexToPixel)0;

    float4x4 preWorldViewProjection = mul (xWorld, xCamerasViewProjection);

    Output.Position =mul(inPos, preWorldViewProjection);
    Output.TexCoords = inTexCoords;
    Output.Normal = normalize(mul(inNormal, (float3x3)xWorld));
    Output.Position3D = mul(inPos, xWorld);

    return Output;
}
```

Since our corresponding pixel shader doesn’t use any matrix constants, we can move on to the second vertex shader, ShadowMapVertexShader. This one has only 2 interesting lines, but is also uses an old matrix: the xLightsWorldViewProjection matrix. This can be constructed by combining the World matrix and the light’s ViewProjection matrix:

```csharp
 SMapVertexToPixel ShadowMapVertexShader( float4 inPos : POSITION)
 {
     SMapVertexToPixel Output = (SMapVertexToPixel)0;

     float4x4 preLightsWorldViewProjection = mul (xWorld, xLightsViewProjection);

     Output.Position = mul(inPos, preLightsWorldViewProjection);
     Output.Position2D = Output.Position;

     return Output;
 }
```

The corresponding pixel shader doesn’t use any matrices, so move on the next and last vertex shader, ShadowedSceneVertexShader. This one needs the WorldViewProjection matrices of both our camera and our light, which we need to calculate. This is done exactly the same as before:

```csharp
SSceneVertexToPixel ShadowedSceneVertexShader( float4 inPos : POSITION, float2 inTexCoords : TEXCOORD0, float3 inNormal : NORMAL)
{
    SSceneVertexToPixel Output = (SSceneVertexToPixel)0;

    float4x4 preWorldViewProjection = mul (xWorld, xCamerasViewProjection);
    float4x4 preLightsWorldViewProjection = mul (xWorld, xLightsViewProjection);

    Output.Position = mul(inPos, preWorldViewProjection);
    Output.Pos2DAsSeenByLight = mul(inPos, preLightsWorldViewProjection);
    Output.Normal = normalize(mul(inNormal, (float3x3)xWorld));
    Output.Position3D = mul(inPos, xWorld);
    Output.TexCoords = inTexCoords;

    return Output;
}
```

When you run this code, you should get the same result as last chapter! Only this time, we’ve passed in only the basic matrices to our effect, and combined them in our shaders. This is a much more scalable approach.

Before ending this chapter, let me give you some proof of my whole story. Using the command prompt, you can let the compiler show you what assembler code your HLSL file would be transformed into. Go to the directory that holds your .fx file, and type:

```csharp
 fxc /Tfx_2_0 OurHLSLfile.fx /Fc:output.fxc
```

In case your .fx file is named OurHLSLfile.fx, this will compile you HLSL code and put the resulting assembler code in the file output.fxc. By default, the fxc compiler will enable preshaders. If you want to see the difference, you can let the compiler know to disable preshaders by using the /Op parameter:

```csharp
 fxc /Op /Tfx_2_0 OurHLSLfile.fx /Fc:output.fxc
```

Below you can file the assembly code of our first vertex shader. Because the total listing would be too large, I snipped out a part of it, but you should get the idea anyway:

![Compiled Shader](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-17Preshaders2.jpg?raw=true)

In the left part, you will see all matrix multiplications are done in the vertex shader itself, which corresponds to the ‘No way’ column of my first image on this page. This way, up to 67 vertex instructions have to be performed for each vertex, which is quite a lot.

In the right part, you’ll see these multiplications have been identified by the compiler and extracted into the preshader. In this case, only 6 shader instructions have to be performed for each vertex!

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-17Preshaders3.jpg?raw=true)

This chapter we haven’t introduced a new XNA technique, but we have discussed an elegant way to clean up the interface between our XNA code and our HLSL code. Our XNA code passes only the basic matrices to the effect, and the HLSL code creates the matrices it needs. This approach is much cleaner and more scalable.

## Our final HLSL code

```csharp
 float4x4 xCamerasViewProjection;
 float4x4 xLightsViewProjection;
 float4x4 xWorld;
 float3 xLightPos;
 float xLightPower;
 float xAmbient;
 
 Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xShadowMap;

sampler ShadowMapSampler = sampler_state { texture = <xShadowMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = clamp; AddressV = clamp;};Texture xCarLightTexture;

sampler CarLightSampler = sampler_state { texture = <xCarLightTexture> ; magfilter = LINEAR; minfilter=LINEAR; mipfilter = LINEAR; AddressU = clamp; AddressV = clamp;};

struct VertexToPixel
{
    float4 Position     : POSITION;    
    float2 TexCoords    : TEXCOORD0;
    float3 Normal        : TEXCOORD1;
    float3 Position3D    : TEXCOORD2;
};

struct PixelToFrame
{
    float4 Color        : COLOR0;
};

float DotProduct(float3 lightPos, float3 pos3D, float3 normal)
{
    float3 lightDir = normalize(pos3D - lightPos);
    return dot(-lightDir, normal);    
}

VertexToPixel SimplestVertexShader( float4 inPos : POSITION0, float3 inNormal: NORMAL0, float2 inTexCoords : TEXCOORD0)
{
    VertexToPixel Output = (VertexToPixel)0;
    
    float4x4 preWorldViewProjection = mul (xWorld, xCamerasViewProjection);
    Output.Position = mul(inPos, preWorldViewProjection);
    Output.TexCoords = inTexCoords;
    Output.Normal = normalize(mul(inNormal, (float3x3)xWorld));    
    Output.Position3D = mul(inPos, xWorld);

    return Output;
}

PixelToFrame OurFirstPixelShader(VertexToPixel PSIn)
{
    PixelToFrame Output = (PixelToFrame)0;    

    float diffuseLightingFactor = DotProduct(xLightPos, PSIn.Position3D, PSIn.Normal);
    diffuseLightingFactor = saturate(diffuseLightingFactor);
    diffuseLightingFactor *= xLightPower;

    PSIn.TexCoords.y--;
    float4 baseColor = tex2D(TextureSampler, PSIn.TexCoords);
    Output.Color = baseColor*(diffuseLightingFactor + xAmbient);

    return Output;
}

technique Simplest
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 SimplestVertexShader();
        PixelShader = compile ps_2_0 OurFirstPixelShader();
    }
}

struct SMapVertexToPixel
{
    float4 Position     : POSITION;
    float4 Position2D    : TEXCOORD0;
};

struct SMapPixelToFrame
{
    float4 Color : COLOR0;
};


SMapVertexToPixel ShadowMapVertexShader( float4 inPos : POSITION)
{
    SMapVertexToPixel Output = (SMapVertexToPixel)0;
    
    float4x4 preLightsWorldViewProjection = mul (xWorld, xLightsViewProjection);

    Output.Position = mul(inPos, preLightsWorldViewProjection);
    Output.Position2D = Output.Position;

    return Output;
}

SMapPixelToFrame ShadowMapPixelShader(SMapVertexToPixel PSIn)
{
    SMapPixelToFrame Output = (SMapPixelToFrame)0;            

    Output.Color = PSIn.Position2D.z/PSIn.Position2D.w;

    return Output;
}


technique ShadowMap
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 ShadowMapVertexShader();
        PixelShader = compile ps_2_0 ShadowMapPixelShader();
    }
}


struct SSceneVertexToPixel
{
    float4 Position             : POSITION;
    float4 Pos2DAsSeenByLight    : TEXCOORD0;
    float2 TexCoords            : TEXCOORD1;
    float3 Normal                : TEXCOORD2;
    float4 Position3D            : TEXCOORD3;
};

struct SScenePixelToFrame
{
    float4 Color : COLOR0;
};

SSceneVertexToPixel ShadowedSceneVertexShader( float4 inPos : POSITION, float2 inTexCoords : TEXCOORD0, float3 inNormal : NORMAL)
{
    SSceneVertexToPixel Output = (SSceneVertexToPixel)0;
    
    float4x4 preWorldViewProjection = mul (xWorld, xCamerasViewProjection);
    float4x4 preLightsWorldViewProjection = mul (xWorld, xLightsViewProjection);

    Output.Position = mul(inPos, preWorldViewProjection);    
    Output.Pos2DAsSeenByLight = mul(inPos, preLightsWorldViewProjection);    
    Output.Normal = normalize(mul(inNormal, (float3x3)xWorld));    
    Output.Position3D = mul(inPos, xWorld);
    Output.TexCoords = inTexCoords;    

    return Output;
}

SScenePixelToFrame ShadowedScenePixelShader(SSceneVertexToPixel PSIn)
{
    SScenePixelToFrame Output = (SScenePixelToFrame)0;    

    float2 ProjectedTexCoords;
    ProjectedTexCoords[0] = PSIn.Pos2DAsSeenByLight.x/PSIn.Pos2DAsSeenByLight.w/2.0f +0.5f;
    ProjectedTexCoords[1] = -PSIn.Pos2DAsSeenByLight.y/PSIn.Pos2DAsSeenByLight.w/2.0f +0.5f;
    
    float diffuseLightingFactor = 0;
    if ((saturate(ProjectedTexCoords).x == ProjectedTexCoords.x) && (saturate(ProjectedTexCoords).y == ProjectedTexCoords.y))
    {
        float depthStoredInShadowMap = tex2D(ShadowMapSampler, ProjectedTexCoords).r;
        float realDistance = PSIn.Pos2DAsSeenByLight.z/PSIn.Pos2DAsSeenByLight.w;
        if ((realDistance - 1.0f/100.0f) <= depthStoredInShadowMap)
        {
            diffuseLightingFactor = DotProduct(xLightPos, PSIn.Position3D, PSIn.Normal);
            diffuseLightingFactor = saturate(diffuseLightingFactor);
            diffuseLightingFactor *= xLightPower;            
            
            float lightTextureFactor = tex2D(CarLightSampler, ProjectedTexCoords).r;
            diffuseLightingFactor *= lightTextureFactor;
        }
    }
        
    float4 baseColor = tex2D(TextureSampler, PSIn.TexCoords);                
    Output.Color = baseColor*(diffuseLightingFactor + xAmbient);

    return Output;
}

technique ShadowedScene
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 ShadowedSceneVertexShader();
        PixelShader = compile ps_2_0 ShadowedScenePixelShader();
    }
}
```

## And our cleaned XNA code

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
 
 namespace XNAseries3
 {
     public struct MyOwnVertexFormat
     {
         public Vector3 position;
         private Vector2 texCoord;
         private Vector3 normal;
 
         public MyOwnVertexFormat(Vector3 position, Vector2 texCoord, Vector3 normal)
         {
             this.position = position;
             this.texCoord = texCoord;
             this.normal = normal;
         }
 
         public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
              (
                  new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
                  new VertexElement(sizeof(float) * 3, VertexElementFormat.Vector2, VertexElementUsage.TextureCoordinate, 0),
                  new VertexElement(sizeof(float) * (3+2), VertexElementFormat.Vector3, VertexElementUsage.Normal, 0)
              );
     }
 
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         GraphicsDeviceManager graphics;
         GraphicsDevice device;
 
         Effect effect;
         Matrix viewMatrix;
         Matrix projectionMatrix;
         VertexBuffer vertexBuffer;
         Vector3 cameraPos;
         Texture2D streetTexture;
         Model lamppostModel;
         Texture2D[] lamppostTextures;
         Model carModel;
         Texture2D[] carTextures;
         Vector3 lightPos;
         float lightPower;
         float ambientPower;
         Matrix lightsViewProjectionMatrix;
         RenderTarget2D renderTarget;
         Texture2D shadowMap;
         Texture2D carLight;
 
         public Game1()
         {
             graphics = new GraphicsDeviceManager(this);
             Content.RootDirectory = "Content";
         }
 
         protected override void Initialize()
         {
             graphics.PreferredBackBufferWidth = 500;
             graphics.PreferredBackBufferHeight = 500;
             graphics.IsFullScreen = false;
             graphics.ApplyChanges();
             Window.Title = "Riemer's XNA Tutorials -- Series 3";
 
             base.Initialize();
         }
 
         protected override void LoadContent()
         {
             device = GraphicsDevice;
 

            effect = Content.Load<Effect> ("OurHLSLfile");            SetUpVertices();
            SetUpCamera();


            streetTexture = Content.Load<Texture2D> ("streettexture");            carModel = LoadModel("car", out carTextures);

            PresentationParameters pp = device.PresentationParameters;
            renderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, true, device.DisplayMode.Format, DepthFormat.Depth24);


            carLight = Content.Load<Texture2D> ("carlight");        }

        private Model LoadModel(string assetName, out Texture2D[] textures)
        {

            Model newModel = Content.Load<Model> (assetName);
            textures = new Texture2D[7];
            int i = 0;
            foreach (ModelMesh mesh in newModel.Meshes)
                foreach (BasicEffect currentEffect in mesh.Effects)
                    textures[i++] = currentEffect.Texture;

            foreach (ModelMesh mesh in newModel.Meshes)
                foreach (ModelMeshPart meshPart in mesh.MeshParts)
                    meshPart.Effect = effect.Clone();

            return newModel;
        }

        private void SetUpVertices()
        {
            MyOwnVertexFormat[] vertices = new MyOwnVertexFormat[18];

            vertices[0] = new MyOwnVertexFormat(new Vector3(-20, 0, 10), new Vector2(-0.25f, 25.0f), new Vector3(0, 1, 0));
            vertices[1] = new MyOwnVertexFormat(new Vector3(-20, 0, -100), new Vector2(-0.25f, 0.0f), new Vector3(0, 1, 0));
            vertices[2] = new MyOwnVertexFormat(new Vector3(2, 0, 10), new Vector2(0.25f, 25.0f), new Vector3(0, 1, 0));
            vertices[3] = new MyOwnVertexFormat(new Vector3(2, 0, -100), new Vector2(0.25f, 0.0f), new Vector3(0, 1, 0));
            vertices[4] = new MyOwnVertexFormat(new Vector3(2, 0, 10), new Vector2(0.25f, 25.0f), new Vector3(-1, 0, 0));
            vertices[5] = new MyOwnVertexFormat(new Vector3(2, 0, -100), new Vector2(0.25f, 0.0f), new Vector3(-1, 0, 0));
            vertices[6] = new MyOwnVertexFormat(new Vector3(2, 1, 10), new Vector2(0.375f, 25.0f), new Vector3(-1, 0, 0));
            vertices[7] = new MyOwnVertexFormat(new Vector3(2, 1, -100), new Vector2(0.375f, 0.0f), new Vector3(-1, 0, 0));
            vertices[8] = new MyOwnVertexFormat(new Vector3(2, 1, 10), new Vector2(0.375f, 25.0f), new Vector3(0, 1, 0));
            vertices[9] = new MyOwnVertexFormat(new Vector3(2, 1, -100), new Vector2(0.375f, 0.0f), new Vector3(0, 1, 0));
            vertices[10] = new MyOwnVertexFormat(new Vector3(3, 1, 10), new Vector2(0.5f, 25.0f), new Vector3(0, 1, 0));
            vertices[11] = new MyOwnVertexFormat(new Vector3(3, 1, -100), new Vector2(0.5f, 0.0f), new Vector3(0, 1, 0));
            vertices[12] = new MyOwnVertexFormat(new Vector3(13, 1, 10), new Vector2(0.75f, 25.0f), new Vector3(0, 1, 0));
            vertices[13] = new MyOwnVertexFormat(new Vector3(13, 1, -100), new Vector2(0.75f, 0.0f), new Vector3(0, 1, 0));
            vertices[14] = new MyOwnVertexFormat(new Vector3(13, 1, 10), new Vector2(0.75f, 25.0f), new Vector3(-1, 0, 0));
            vertices[15] = new MyOwnVertexFormat(new Vector3(13, 1, -100), new Vector2(0.75f, 0.0f), new Vector3(-1, 0, 0));
            vertices[16] = new MyOwnVertexFormat(new Vector3(13, 21, 10), new Vector2(1.25f, 25.0f), new Vector3(-1, 0, 0));
            vertices[17] = new MyOwnVertexFormat(new Vector3(13, 21, -100), new Vector2(1.25f, 0.0f), new Vector3(-1, 0, 0));

            vertexBuffer = new VertexBuffer(device, MyOwnVertexFormat.VertexDeclaration, vertices.Length, BufferUsage.WriteOnly);
            vertexBuffer.SetData(vertices);
        }

        private void SetUpCamera()
        {
            cameraPos = new Vector3(-25, 13, 18);
            viewMatrix = Matrix.CreateLookAt(cameraPos, new Vector3(0, 2, -12), new Vector3(0, 1, 0));
            projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 200.0f);
        }

        protected override void UnloadContent()
        {
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                this.Exit();

            UpdateLightData();

            base.Update(gameTime);
        }

        private void UpdateLightData()
        {
            ambientPower = 0.2f;

            lightPos = new Vector3(-18, 5, -2);
            lightPower = 2f;

            Matrix lightsView = Matrix.CreateLookAt(lightPos, new Vector3(-2, 3, -10), new Vector3(0, 1, 0));
            Matrix lightsProjection = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver2, 1f, 5f, 100f);

            lightsViewProjectionMatrix = lightsView * lightsProjection;
        }


        protected override void Draw(GameTime gameTime)
        {
            device.SetRenderTarget(renderTarget);            
            device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);

            DrawScene("ShadowMap");

            device.SetRenderTarget(null);
            shadowMap = (Texture2D)renderTarget;

            device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);            
            DrawScene("ShadowedScene");
            shadowMap = null;

            base.Draw(gameTime);
        }

        private void DrawScene(string technique)
        {
            effect.CurrentTechnique = effect.Techniques[technique];
            effect.Parameters["xCamerasViewProjection"].SetValue(viewMatrix * projectionMatrix);
            effect.Parameters["xLightsViewProjection"].SetValue(lightsViewProjectionMatrix);
            effect.Parameters["xWorld"].SetValue(Matrix.Identity);
            effect.Parameters["xTexture"].SetValue(streetTexture);
            effect.Parameters["xLightPos"].SetValue(lightPos);
            effect.Parameters["xLightPower"].SetValue(lightPower);
            effect.Parameters["xAmbient"].SetValue(ambientPower);
            effect.Parameters["xShadowMap"].SetValue(shadowMap);
            effect.Parameters["xCarLightTexture"].SetValue(carLight);

            foreach (EffectPass pass in effect.CurrentTechnique.Passes)
            {
                pass.Apply();

                device.SetVertexBuffer(vertexBuffer);
                device.DrawPrimitives(PrimitiveType.TriangleStrip, 0, 16);
            }

            Matrix car1Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi) * Matrix.CreateTranslation(-3, 0, -15);
            DrawModel(carModel, carTextures, car1Matrix, technique);

            Matrix car2Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi * 5.0f / 8.0f) * Matrix.CreateTranslation(-28, 0, -1.9f);
            DrawModel(carModel, carTextures, car2Matrix, technique);
        }

        private void DrawModel(Model model, Texture2D[] textures, Matrix wMatrix, string technique)
        {
            Matrix[] modelTransforms = new Matrix[model.Bones.Count];
            model.CopyAbsoluteBoneTransformsTo(modelTransforms);
            int i = 0;
            foreach (ModelMesh mesh in model.Meshes)
            {
                foreach (Effect currentEffect in mesh.Effects)
                {
                    Matrix worldMatrix = modelTransforms[mesh.ParentBone.Index] * wMatrix;
                    currentEffect.CurrentTechnique = currentEffect.Techniques[technique];
                    currentEffect.Parameters["xCamerasViewProjection"].SetValue(viewMatrix * projectionMatrix);
                    currentEffect.Parameters["xLightsViewProjection"].SetValue(lightsViewProjectionMatrix);
                    currentEffect.Parameters["xWorld"].SetValue(worldMatrix);
                    currentEffect.Parameters["xTexture"].SetValue(textures[i++]);
                    currentEffect.Parameters["xLightPos"].SetValue(lightPos);
                    currentEffect.Parameters["xLightPower"].SetValue(lightPower);
                    currentEffect.Parameters["xAmbient"].SetValue(ambientPower);
                    currentEffect.Parameters["xShadowMap"].SetValue(shadowMap);
                    currentEffect.Parameters["xCarLightTexture"].SetValue(carLight);
                }
                mesh.Draw();
            }
        }

    }
}
```
