# Transforming vertices to texture space using Projective Texturing

Now we have a Shadow Map, it’s time to draw the scene as seen by our camera, and check whether the pixels should be shadowed or lit. This can be decided by sampling our Shadow Map at the correct location. As goal for this chapter, we’ll be drawing the scene from our camera’s point of view, with the color of each pixel being the color stored in the Shadow Map for that pixel.

First of all, this means we’ll be drawing our scene using 2 different techniques: the first one will generate our Shadow Map which we store in a texture, the second one will draw the scene to the screen again. You can already add this second technique to your HLSL file:

```csharp
technique ShadowedScene
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 ShadowedSceneVertexShader();
        PixelShader = compile ps_2_0 ShadowedScenePixelShader();
    }
}
```

Because we need 2 brand new shader methods, it’s good practice to define new structs that hold the output data of both methods:

```csharp
struct SSceneVertexToPixel
{
    float4 Position             : POSITION;
    float4 Pos2DAsSeenByLight    : TEXCOORD0;
};

struct SScenePixelToFrame
{
    float4 Color : COLOR0;
};
```

You can see the vertex shader will pass the (required) 2D position as seen by the camera to the pixel shader, as well as the 2D position as seen by the light. From the latter, we can retrieve the position of the Shadow Map that corresponds to the current pixel that is being drawn. The pixel shader will again only output the pixel’s color.

Our pixel shader will be using the Shadow Map, which can be treated as a standard texture:

```csharp
Texture xShadowMap;

sampler ShadowMapSampler = sampler_state { texture = <xShadowMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = clamp; AddressV = clamp;};
```

Let’s start by coding the vertex shader. A quick reminder: this second technique has to draw the scene, as seen by the camera. When this second technique is being called, the first technique has already finished drawing the shadow map. Important: this shadow map was drawn as seen by the headlights, at the bottom left of our scene. So now, for every pixel seen by the camera, we have to find which part of shadow map corresponds to this pixel. In other words, we need the 2D screen coordinates of the shadow map that correspond with every pixel being drawn. Because this can be somewhat difficult to grasp, I’ve added a small picture below:

![Projection](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-14Texturing1.jpg?raw=true)

I’ve indicated the points of interest with a red dot. Say we want to find the color of the dotted pixel of the wall. What we want to know is the 2D coordinate of the corresponding pixel in our Shadow Map.

The first step would be to project the 3D coordinates of the red-dotted pixel of the wall to 2D camera space of the light. This can be done exactly the same way as we’ve done in the ShadowMap technique: by transforming the 3D positions with the WorldViewProjection matrix of the light. The resulting 2D coordinate is stored in Output.Pos2DAsSeenByLight:

```csharp
SSceneVertexToPixel ShadowedSceneVertexShader( float4 inPos : POSITION)
{
    SSceneVertexToPixel Output = (SSceneVertexToPixel)0;

    Output.Position = mul(inPos, xWorldViewProjection);
    Output.Pos2DAsSeenByLight= mul(inPos, xLightsWorldViewProjection);
    return Output;
}
```

Now our pixel shader has access to the homogeneous screen coordinates of our light. The 4th (=homogeneous) coordinate has already been discussed 2 chapters earlier, but it remains quite a difficult line. We have a 4th coordinate, since the value stored in Output.Pos2DAsSeenByLightis the result of a multiplication with a 4x4 matrix. Before we can use the X,Y and Z components of such a result, we need to divide them by the W component. If you’re interested why, you can check out my articles on matrices you can find in the ‘Extra Reading’ section at the left of the site. For now, remember we need to divide by the W component. Because screen coordinates are 2D, we’re not using the Z component. So the difficult line reduces to: ‘divide X and Y by W’.

When we do this, we would get a 2D screen coordinate, with the X and Y component between the [-1, 1] region. The coord (-1,-1) would represent the upper left corner, whereas coord (0,1) would represent the pixel in the middle of the right.

Remember texture coordinates have to be between the [0, 1] region, so we need a simple remap: point (-1,-1) has to become (0,0), while point (1,1) has to stay (1,1). This is what we need:

![Formula](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/?raw=true)

Most people use the ‘Projective matrix’ to do this, sometimes because they don’t know what it’s doing :) For a change, we’re simply going to code this formula in our pixel shader:

```csharp
SScenePixelToFrame ShadowedScenePixelShader(SSceneVertexToPixel PSIn)
{
    SScenePixelToFrame Output = (SScenePixelToFrame)0;

    float2 ProjectedTexCoords;
    ProjectedTexCoords[0] = PSIn.Pos2DAsSeenByLight.x/PSIn.Pos2DAsSeenByLight.w/2.0f +0.5f;
    ProjectedTexCoords[1] = -PSIn.Pos2DAsSeenByLight.y/PSIn.Pos2DAsSeenByLight.w/2.0f +0.5f;

    Output.Color = tex2D(ShadowMapSampler, ProjectedTexCoords);

    return Output;
}
```

You see we receive the homogeneous 2D screen coordinates as input to our pixel shader, and derive the 2D texture coordinates from them. We sample the value of our Shadow map at that position and set this as output color for the pixel. Because this all is not that trivial, you can find some extra discussions with sample code in the forum of this chapter.

That’s it for the HLSL code, now let’s switch over to the XNA code. Because at the moment we have 2 techniques, we have to draw our scene twice. All you have to do is add one line to the end of your Draw method:

```csharp
 protected override void Draw(GameTime gameTime)
 {
     device.SetRenderTarget(0, renderTarget);
 
     device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
     DrawScene("ShadowMap");
 
     device.SetRenderTarget(0, null);
     shadowMap = renderTarget.GetTexture();
 
     device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
     DrawScene("ShadowedScene");     shadowMap = null;
 
 
     base.Draw(gameTime);
 }
```

First, the custom render target is activated and cleared. The scene is rendered into this target using the ShadowMap technique, as seen by the headlights of the car. This target is deactivated by re-activating the backbuffer for the screen and its contents is saved into the shadowMap texture. Next, the backbuffer for the screen is cleared and the whole scene is rendered again, this time using the ShadowedScene technique as seen by our camera. At the very end, you let go the shadowMap, as otherwise XNA will complain the next frame when it wants to write into the shadowMap while it’s in use.

OK, when you run this code, you get a nice black screen, which is because we haven’t yet set the xShadowMap variable of the effects! So add this line of code for the trianglestrip, which we should now put in the DrawScene:

```csharp
 effect.Parameters["xShadowMap"].SetValue(shadowMap);
```

And for our models, in the DrawModel method:

```csharp
 currentEffect.Parameters["xShadowMap"].SetValue(shadowMap);
```

Now when you run the code, you should see something like the image below. It looks like our normal scene, except for the colors: these are taken from the Shadow map. When you look at the car, you see it has the same color as in the Shadow map. But when you look at the part of the wall behind the car, you can see it has the same color as the car! Because of this, next chapter we’ll be able to detect the shadowed areas.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-14Texturing3.jpg?raw=true)

This Projective texturing technique is the last mathematical part of the Series, so there’s no reason to start panicking. But with the explanation found in this chapter, you should at least know why/what you are doing, and with my articles on matrices in the Extra Reading section you should even be able to understand the maths behind it. Remember you can always post questions in the forum for further explanation.

## The complete HLSL code

```csharp
float4x4 xWorldViewProjection;
float4x4 xLightsWorldViewProjection;
float4x4 xWorld;
float3 xLightPos;
float xLightPower;
float xAmbient;

Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
 Texture xShadowMap;

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
 };
 
 struct SScenePixelToFrame
 {
     float4 Color : COLOR0;
 };
 
 SSceneVertexToPixel ShadowedSceneVertexShader( float4 inPos : POSITION)
 {
     SSceneVertexToPixel Output = (SSceneVertexToPixel)0;
 
     Output.Position = mul(inPos, xWorldViewProjection);    
     Output.Pos2DAsSeenByLight= mul(inPos, xLightsWorldViewProjection);    
     return Output;
 }
 
 SScenePixelToFrame ShadowedScenePixelShader(SSceneVertexToPixel PSIn)
 {
     SScenePixelToFrame Output = (SScenePixelToFrame)0;    
 
     float2 ProjectedTexCoords;
     ProjectedTexCoords[0] = PSIn.Pos2DAsSeenByLight.x/PSIn.Pos2DAsSeenByLight.w/2.0f +0.5f;
     ProjectedTexCoords[1] = -PSIn.Pos2DAsSeenByLight.y/PSIn.Pos2DAsSeenByLight.w/2.0f +0.5f;
 
     Output.Color = tex2D(ShadowMapSampler, ProjectedTexCoords);
 
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

## Here’s our XNA code

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
            lightPower = 1.0f;

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

[Adding shadows](Riemers3DXNA3hlsl15realshadow)
