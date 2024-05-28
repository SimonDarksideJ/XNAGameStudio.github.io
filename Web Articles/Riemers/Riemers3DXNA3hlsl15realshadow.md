# Adding shadows to our colored scene

Now we can sample the correct position in our Shadow map for each pixel in our scene, we have all the ingredients we need to create our shadows. We will be drawing our scene with real colors and lighting like we did before, so our vertex shader will need to pass the texture coordinates, the normal and the 3D position of every vertex to the interpolator and pixel shader. So we can add these 3 variables to our SSceneVertexToPixel struct:

```csharp
float2 TexCoords            : TEXCOORD1;
float3 Normal                : TEXCOORD2;
float4 Position3D            : TEXCOORD3;
```

We’ve already seen how to fill generate these values in the vertex shader in the previous chapters.

This chapter, we’ll also test the real distance between the pixel and the light against the value found in the Shadow Map. As done in the chapter where we created the shadow map, this distance can be found from the 2D position as seen by the light, by dividing its Z components by its homogeneous component. Our pixel shader already has access to this 2D position, as it’s stored in the Pos2DasSeenByLight variable.

Remember we need again all info sent to us in the vertex stream, so pay special attention to the first line:

```csharp
SSceneVertexToPixel ShadowedSceneVertexShader( float4 inPos : POSITION, float2 inTexCoords : TEXCOORD0, float3 inNormal : NORMAL)
{
    SSceneVertexToPixel Output = (SSceneVertexToPixel)0;

    Output.Position = mul(inPos, xWorldViewProjection);
    Output.Pos2DAsSeenByLight = mul(inPos, xLightsWorldViewProjection);
    Output.Normal = normalize(mul(inNormal, (float3x3)xWorld));
    Output.Position3D = mul(inPos, xWorld);
    Output.TexCoords = inTexCoords;

    return Output;
}
```

This vertex shader contains nothing we haven’t seen before, so let’s move on to the pixel shader. This chapter, we’ll only draw the pixels that are being lit by the headlights. This means we first have to check if the pixels are in view of the headlights. In other words: if the projected x and y coordinates are within the [0, 1] range. For this, we can use the HLSL ‘saturate’ method, which clips any value to this range. So if the value after clipping is not the same as the value before clipping, we know the value wasn’t in the [0, 1] range, and thus isn’t in the view of our headlights. In HLSL code:

```csharp
float diffuseLightingFactor = 0;
if ((saturate(ProjectedTexCoords).x == ProjectedTexCoords.x) && (saturate(ProjectedTexCoords).y == ProjectedTexCoords.y))
{
}

float4 baseColor = tex2D(TextureSampler, PSIn.TexCoords);
    if (xSolidBrown == true)
        baseColor = float4(0.25f, 0.21f, 0.20f, 1);

Output.Color = baseColor*(diffuseLightingFactor + xAmbient);
```

I agree, this code does a little more than discussed above. First we set the impact of the light to 0. The if structure checks whether the current pixel is in sight of the light, as discussed above. If this is true, we will change the impact of the light, which we will do next. If the pixel is not in the sight of the light, the impact of the light will remain 0 and the scene will be rendered with only the ambient light.

So if the pixel successfully enters this if-block, it can be lit by the light. Next, we will check whether the pixel isn’t shadowed by another object. For this, we first retrieve the distance between pixel and light, as stored in the Shadow Map:

```csharp
float depthStoredInShadowMap = tex2D(ShadowMapSampler, ProjectedTexCoords).r;
```

Next, we calculate the real distance between the pixel and the light, which is done exactly the same as in the chapter you made the shadow map:

```csharp
float realDistance = PSIn.Pos2DAsSeenByLight.z/PSIn.Pos2DAsSeenByLight.w;
```

Now we check if the real distance isn’t larger than the value stored in the shadow map:

```csharp
if ((realDistance - 1.0f/100.0f) <= depthStoredInShadowMap)
{
}
```

You see we subtracted a little bias of 1/100. Let me show you why. We store depth information as a color in our Shadow Map. One color component of this map is stored in only 8 bits, so the smallest difference is 1/256. Now say the maximal distance in our scene is 40.0f, so the smallest difference is 40/256 = 0.156f. This means that in even the best case, all points with distances 0.156f to 0.234f will be stored as 0.156f.

Take for example a point at real distance 2.0f (which is stored as 0.156f in our shadow map!). You would like to check whether the real and stored distances are the same: if (2.0f == 0.156f) so this would FAIL, and you would think the point is in the shadow of another object. For this, we need to subtract a small value of our real distance while comparing

OK, with that out of the way, we can move on. If the real distance equals the distance stored in the shadow map, the pixel should be lit. This is done by changing the diffuseLightingFactor value. This is done exactly like we did a few chapters ago: by calculating the dot-product-factor and the power of the light. So we get as pixel shader:

```csharp
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
        }
    }

    float4 baseColor = tex2D(TextureSampler, PSIn.TexCoords);
    Output.Color = baseColor*(diffuseLightingFactor + xAmbient);

    return Output;
}
```

Now we’ve had the HLSL part for this chapter, let’s move on to the XNA code.

Because we’re only working with one light, let’s make it a bit more powerful:

```csharp
 lightPower = 2f;
```

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-15Shadows1.jpg?raw=true)

Now try to run this code. Finally, we see the first shadows! The car casts a nice shadow. Note that only the pixels in the view of the headlights are lit brightly, the others are only visible thanks to the ambient lighting component (this is obtained by the if ((saturate(ProjectedTexCoords.x) ==…. Check in the pixel shader).

Although we’ve got shadows, our light itself hasn’t got a nice shape: you can see the corners of its view frustum. Next chapter we’re going to solve this.

> You can try these exercises to practice what you've learned:
>
> - Open the shaping image in paint, and add some gray areas. Load this image in XNA and look at the impact.

## Our updated HLSL code

```csharp
float4x4 xWorldViewProjection;
float4x4 xLightsWorldViewProjection;
float4x4 xWorld;
float3 xLightPos;
float xLightPower;
float xAmbient;

Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xShadowMap;

sampler ShadowMapSampler = sampler_state { texture = <xShadowMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = clamp; AddressV = clamp;};

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
    
    Output.Position =mul(inPos, xWorldViewProjection);
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

    Output.Position = mul(inPos, xLightsWorldViewProjection);
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
 
     Output.Position = mul(inPos, xWorldViewProjection);    
     Output.Pos2DAsSeenByLight = mul(inPos, xLightsWorldViewProjection);    
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
        }

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
             effect.Parameters["xWorldViewProjection"].SetValue(Matrix.Identity * viewMatrix * projectionMatrix);
             effect.Parameters["xTexture"].SetValue(streetTexture);
             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
             effect.Parameters["xLightPos"].SetValue(lightPos);
             effect.Parameters["xLightPower"].SetValue(lightPower);
             effect.Parameters["xAmbient"].SetValue(ambientPower);
             effect.Parameters["xLightsWorldViewProjection"].SetValue(Matrix.Identity * lightsViewProjectionMatrix);
             effect.Parameters["xShadowMap"].SetValue(shadowMap);
 
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
                     currentEffect.Parameters["xWorldViewProjection"].SetValue(worldMatrix * viewMatrix * projectionMatrix);
                     currentEffect.Parameters["xTexture"].SetValue(textures[i++]);
                     currentEffect.Parameters["xWorld"].SetValue(worldMatrix);
                     currentEffect.Parameters["xLightPos"].SetValue(lightPos);
                     currentEffect.Parameters["xLightPower"].SetValue(lightPower);
                     currentEffect.Parameters["xAmbient"].SetValue(ambientPower);
                     currentEffect.Parameters["xLightsWorldViewProjection"].SetValue(worldMatrix * lightsViewProjectionMatrix);
                     currentEffect.Parameters["xShadowMap"].SetValue(shadowMap);
                 }
                 mesh.Draw();
             }
         }
 
     }
 }
```

## Next Steps

[Changing the shape of our light](Riemers3DXNA3hlsl16shapingthelight)
