# Creating our first light

This chapter we’ll create a point light. This is a point in 3D space, which shines light in every direction. Every object is lit by an amount of light, which depends on the angle between the normal and the direction of the light. This is found by taking the dot product between that object’s normal and the direction of the incoming light. If you want a picture to illustrate this, have a look at ‘XNA light basics’ in Series 1.

> This chapter immediately discusses per-pixel lighting. For a full, step-by-step explanation of lighting in XNA and the dot product, read Chapter 6 on lighting.

At this moment we have normal data included in our vertex stream, so we can go straight to the HLSL code. First, let’s have a look at how we can light our quads:

![Normals](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-11Light1.jpg?raw=true)

In the left quad, the dot product for every vertex is calculated by the vertex shader. Imagine our light is exactly above the centre of the quad. In that case, the angle between the direction (the thin blue lines) of the light and the plane is the same in every vertex. In that case, let’s say the dot product between this light direction and the normal is 0.5 for all four vertices. So 0.5 would be the output of the vertex shader for each of the 4 vertices.

Now the interpolator comes into play. For each pixel of our quad, it interpolates this value, and sends the interpolated value to the pixel shader. In this case, it’s very easy: the interpolated value is 0.5 for each pixel! This means it will seem as if every pixel in our quad is lit the same way, which is wrong.

The right quad illustrates what should happen. For each pixel, the dot product has to be calculated separately: each pixel will get its correct value. For example, the corner points will still get value 0.5, while the pixel exactly below the light will get value 1.0, as the direction of the normal is exactly along the direction of the light.

So now you’re completely convinced the pixel shader is the way to go, let’s start by creating a method that calculates the dot product, if you give it the 3D position of the light, the 3D position of the pixel and the normal in that pixel. This method will be called by your pixel shader method, so put it for example between your vertex and pixel shader.

```csharp
float DotProduct(float3 lightPos, float3 pos3D, float3 normal)
{
    float3 lightDir = normalize(pos3D - lightPos);
    return dot(-lightDir, normal);
}
```

First the direction of the light towards the pixel is calculated, this is the vector between the 3D position of the light and the 3D position of the pixel. Then we calculate the dot product between this light direction and the normal in the pixel, which is what the method returns. Note that the direction of the light needs to be negated, as this direction and the normal have the opposite direction.

We will call the DotProduct from within our pixel shader, as discussed above. You can see this method requires the 3D position as well as the normal to be available to the pixel shader, so we need to update our VertexToPixel structure:

```csharp
struct VertexToPixel
{
    float4 Position     : POSITION;
    float2 TexCoords    : TEXCOORD0;
    float3 Normal        : TEXCOORD1;
    float3 Position3D    : TEXCOORD2;
};
```

Once again, we’re using the TEXCOORDn semantic to pass values from our vertex shader to our pixel shader. Next, let’s change our vertex shader method so it actually uses the normal data we’ve been sending in the vertex stream since the previous chapter:

VertexToPixel SimplestVertexShader( float4 inPos : POSITION0, float3 inNormal: NORMAL0, float2 inTexCoords : TEXCOORD0)

Now it’s time to fill the Output.Normal and Output.Position3D values in the vertex shader. The normal, however, needs some more explanation. Think of our meshes: they contain normal data. However, before we actually draw the meshes, we have rotated them and translated them. This means we also need to rotate the normals. We must not translate them (because the length of a normal should always be 1, rotating a normal will also give a normal of length 1. This means, that if we translate a normal over 10 units to the left, all of the normals of the model will points to the left! See Recipe 6-5).

Currently, we only pass the WorldViewProjection matrix to our effect. From this matrix, it is impossible to reconstruct rotation data, so we will also pass the World matrix:

```csharp
float4x4 xWorld;
```

This 4x4 matrix contains rotation, translation and scaling information. However, if we cast this 4x4 matrix to a 3x3 matrix, we will only retain the rotation information! The next 2 lines fill the output of the vertex shader:

```csharp
Output.Normal = normalize(mul(inNormal, (float3x3)xWorld));
Output.Position3D = mul(inPos, xWorld);
```

You see the normal is multiplied by the 3x3 world matrix (containing only rotation data). This gives you the rotated version of the normal. Then, it is normalized, so we make sure the length becomes 1 again.

I guess the second line requires no explanation, as the 3D position of the vertices is passed straight on to the interpolator, which interpolates the 3D position for each pixel. Note that this needs to be transformed by the World matrix, otherwise for example the positions of both lampposts would be the same.

So far for the vertex shader, but before we move on to the pixel shader, let’s define 3 more variables. The first holds the position of the light, the second one allows the XNA application to set the strength of the light and the third one will set the ambient light, present in each and every corner of our 3D scene. Since these variables are the same for all vertices and pixel of one frame, they should be specified by XNA through XNA-to-HLSL variables:

```csharp
float3 xLightPos;
float xLightPower;
float xAmbient;
```

Let’s move on to our pixel shader. This is the contents of the PSIn structure your pixel shader receives from the interpolator:

- PSIn.Position : the 2D position of the current pixel in screen coordinates; remember our pixel shader can NOT use this data!

- PSIn.TexCoords : the 2D coordinates indicating the position where to sample the texture

- PSIn.Normal : the direction of the normal in the current pixel

- PSIn.Position3D : the 3D coordinate of the current pixel

It’s time to start updating our pixel shader. These should be your first 4 lines:

```csharp
PixelToFrame Output = (PixelToFrame)0;

float diffuseLightingFactor = DotProduct(xLightPos, PSIn.Position3D, PSIn.Normal);
diffuseLightingFactor = saturate(diffuseLightingFactor);
diffuseLightingFactor *= xLightPower;
```

The second line calls our DotProduct method for the current pixel. As a result, we obtain a value which indicates the amount of light that is caught, and thus reflected by the current pixel. Because the result of a dot product is always within the [-1, 1] range (if both vectors are normalized), we first need to bring it to the [0,1] range using the saturate intrinsic. Next, we multiply it by the strength of the light.

For now, we will simply display this value as color:

```csharp
Output.Color = diffuseLightingFactor;
```

That’s it for the HLSL code! When you run the code this time, you shouldn’t get any errors. However, your screen will look disappointingly black. This is because we still have to set all the XNA-to-HLSL variables from within XNA.

So go to our XNA code, where we’ll be adding 3 variables to our code:

```csharp
 Vector3 lightPos;
 float lightPower;
 float ambientPower;
```

Next, add a small method that sets these values:

```csharp
 private void UpdateLightData()
 {
     lightPos = new Vector3(-10, 4, -2);
     lightPower = 1.0f;
     ambientPower = 0.2f;
 }
```

The benefit of this, is that you can call it from the Update method, to have the position or strength of your light changed every frame. So don’t forget to call this method from our Update method:

```csharp
 UpdateLightData();
```

Although now the method will set 3 constant values, feel free to make them change every frame. We of course need to pass these values, together with the World matrix to our effect file. Let’s first add this to our Draw method, for our trianglestrip:

```csharp
 effect.Parameters["xWorld"].SetValue(Matrix.Identity);
 effect.Parameters["xLightPos"].SetValue(lightPos);
 effect.Parameters["xLightPower"].SetValue(lightPower);
 effect.Parameters["xAmbient"].SetValue(ambientPower);
```

And also for our models, in the DrawModel method. Notice that here we need to set the parameters of the ‘currenteffect’, instead of the ‘effect’, and pass in the correct World matrix for the current part of the Model:

```csharp
 currentEffect.Parameters["xWorld"].SetValue(worldMatrix);
 currentEffect.Parameters["xLightPos"].SetValue(lightPos);
 currentEffect.Parameters["xLightPower"].SetValue(lightPower);
 currentEffect.Parameters["xAmbient"].SetValue(ambientPower);
```

Now when you run this code, you should finally see the image below. It indicates how much light is falling on each pixel. You can see clearly where the light is positioned. Notice that we’re not yet taking the distance into account; the only reason why the end of the wall gets less illuminated is because the angle to the light is getting sharper.

![Alpha](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-11Light2.jpg?raw=true)

Of course we don’t want a grayscale image; it’s just a debugging image. What we want are colors, and this code suffices:

```csharp
float4 baseColor = tex2D(TextureSampler, PSIn.TexCoords);
Output.Color = baseColor*(diffuseLightingFactor + xAmbient);
```

First we find the base color. This has to be sampled from the texture in case of the cars and street.

Once you know the base color, you multiply it with the combination of the light that will be different in each pixel, and the ambient light which is present in our entire scene!

When you compile the HLSL code and run your XNA app again, you should see this image:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-11Light3.jpg?raw=true)

So now we have the basics of lighting in our HLSL code, what’s next? This is where things get interesting: we’ll be starting our shadowing code.

> You can try these exercises to practice what you've learned:
>
> - Play around with the position and strength of your light, as well as with the ambient light setting.
> - See what these changes do to your graylevel map.

## Our HLSL code

```csharp
float4x4 xWorldViewProjection;

 float4x4 xWorld;
 float3 xLightPos;
 float xLightPower;
 float xAmbient;


Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
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
```

## And our XNA code

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
            lamppostModel = LoadModel("lamppost", out lamppostTextures);
            lamppostTextures[0] = streetTexture;
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
            lightPos = new Vector3(-10, 4, -2);
            lightPower = 1.0f;
            ambientPower = 0.2f;
        }

        protected override void Draw(GameTime gameTime)
        {
            device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);

            effect.CurrentTechnique = effect.Techniques["Simplest"];
            effect.Parameters["xWorldViewProjection"].SetValue(Matrix.Identity * viewMatrix * projectionMatrix);
            effect.Parameters["xTexture"].SetValue(streetTexture);

             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
             effect.Parameters["xLightPos"].SetValue(lightPos);
             effect.Parameters["xLightPower"].SetValue(lightPower);
             effect.Parameters["xAmbient"].SetValue(ambientPower);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.SetVertexBuffer(vertexBuffer);
                 device.DrawPrimitives(PrimitiveType.TriangleStrip, 0, 16);
             }
 
             Matrix car1Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi) * Matrix.CreateTranslation(-3, 0, -15);
             DrawModel(carModel, carTextures, car1Matrix, "Simplest");
 
             Matrix car2Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi * 5.0f / 8.0f) * Matrix.CreateTranslation(-28, 0, -1.9f);
             DrawModel(carModel, carTextures, car2Matrix, "Simplest");
 
             base.Draw(gameTime);
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
                 }
                 mesh.Draw();
             }
         }
 
     }
 }
```

## Next Steps

[Shadow Mapping](Riemers3DXNA3hlsl12shadowmap)
