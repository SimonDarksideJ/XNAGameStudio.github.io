# Explanation of the Shadow Mapping algorithm

Now we know some HLSL basics and have defined our first light, it’s time for something more complex. Now you’ve added the light to your scene, you should have noted that the cars and lampposts don’t cast any shadows on the wall or pavement.

Let’s see how we can add real shadowing to our scene. We want this shadowing algorithm to be completely dynamic: we want to define the position and direction of our light only one time, and every object within its range should automatically cast a shadow.

Let’s start with the car at the bottom left corner of our screen. Its headlights are shining light towards the right side of our scene. So the light hits the lampposts and the other car, which have to cast shadows on the wall. The question: how do we know which pixels on the wall are lit, and which are shadowed?

While I explain the Depth Mapping algorithm, you can have a look at the image below, where the 2 major steps are illustrated. The first step would be to draw the scene, as seen by the headlights. This means we have to move our camera to the position of the headlights. Using this point of view, the only thing we are interested in is the distance of every pixel to the headlights. For example, the first lamppost would be 4 meters away of the headlights. Very important: the pixels of the wall behind this first lamppost are not seen by the headlights. At the location of these pixels on the wall, 4 meters was stored as distance to the headlights. We store this distance information of the whole scene in what is called a shadow map or depth map.

During the second phase, we draw the scene the usual way; that is, from our camera’s point of view. Only this time, for each pixel we first calculate the distance to the headlights, and compare this depth to the depth stored in the depth map. For most objects, both distances will be the same. Our lamppost, for example, will still be 4 meters away from the headlights.

However, when we calculate the distance for the pixels of the wall behind the light, we find that the distance to the headlights is 5 meters. This is not the same as the distance of 4 meters that was stored for these pixels. This way, we know these pixels can not be seen by the headlights, and should therefore not be lit.

![Shadow](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-12Shadow1.jpg?raw=true)

This will all be better illustrated as we move on to the first step: drawing the depth map. We’ll be defining a new technique for this in our HLSL code, ShadowMap, that renders the depth of the scene to the screen. You can already add the technique definition to the very end of our HLSL file:

```csharp
technique ShadowMap
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 ShadowMapVertexShader();
        PixelShader = compile ps_2_0 ShadowMapPixelShader();
    }
}
```

Our vertex shader will be very simple, as this is the only output it needs to generate. Add this immediately above the technique definition:

```csharp
struct SMapVertexToPixel
{
    float4 Position     : POSITION;
    float4 Position2D    : TEXCOORD0;
};
```

As always, we need to supply the interpolator and pixel shader with the 2D screen coordinates. Only this time, since this screen coordinate also contains the distance between the object and the camera, we need this information also in our pixel shader, so we need to pass it using one of the TEXCOORDn semantics. We’ve already covered the biggest part of this, so this is our new vertex shader:

```csharp
SMapVertexToPixel ShadowMapVertexShader( float4 inPos : POSITION)
{
    SMapVertexToPixel Output = (SMapVertexToPixel)0;

    Output.Position = mul(inPos, xLightsWorldViewProjection);
    Output.Position2D = Output.Position;

    return Output;
}
```

Something very important to notice here: we’re using xLightsWorldViewProjection here instead of xWorldViewProjection, because this time we need to look at the scene as seen by the headlights, instead of as by our camera.

The WorldViewProjection matrix is the combination of 3 matrices (World, View and Projection matrix). Although its name might confuse you, the LightsWorldViewProjection matrix is a combination of also 3 matrices: the same World matrix, but a View and Projection matrix specific to the light. These View and Projection matrices are different from those of the camera.

This is a new matrix, so we need to initialize it at the top of our HLSL code:

```csharp
float4x4 xLightsWorldViewProjection;
```

Which we’ll fill from within our XNA app later on in this chapter. Next, we’ll code our pixel shader, which again only has to calculate the color:

```csharp
struct SMapPixelToFrame
{
    float4 Color : COLOR0;
};
```

And this will be our pixel shader:

```csharp
SMapPixelToFrame ShadowMapPixelShader(SMapVertexToPixel PSIn)
{
    SMapPixelToFrame Output = (SMapPixelToFrame)0;            

    Output.Color = PSIn.Position2D.z/PSIn.Position2D.w;

    return Output;
}
```

The X and Y coordinates of the PSIn.Position contain the X and Y values of the screen coordinate of the current pixel, but the Z coordinate is also very useful as it contains the distance between the camera and the pixel.

However, this vector is the result of a multiplication of a vector and a 4x4 matrix, which happened in the vertex shader. the result of such a multiplication has 4 components: X,Y,Z and W. We cannot use any of the X,Y or Z components immediately, we first need to divide them by the W component. The W component is called the “homogeneous” component, you can find more explanation on this in the “Extra Reading” section of my site.

After dividing the Z component by the homogeneous component, the result will be between 0 and 1, where 0 corresponds to pixels at the near clipping plane and 1 to pixels at the far clipping plane, as defined in the creation of the Projection matrix.

So far for the HLSL part. Let’s go to our XNA code, where we need to pass all variables to our effect. Let’s start with the xLightsWorldViewProjection variable. Because this depends on the position of the light, we’ll update it in the UpdateLightData method, and we need to add this variable to our code:

```csharp
 Matrix lightsViewProjectionMatrix;
```

Now we’ll change the contents of our UpdateLightData method. We change the position of our light to the front of the car, change its intensity and the 3rd line is new:

```csharp
 private void UpdateLightData()
 {
     ambientPower = 0.2f;

     lightPos = new Vector3(-18, 5, -2);
     lightPower = 1.0f;

     Matrix lightsView = Matrix.CreateLookAt(lightPos, new Vector3(-2, 3, -10), new Vector3(0, 1, 0));
     Matrix lightsProjection = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver2, 1f, 5f, 100f);

     lightsViewProjectionMatrix = lightsView* lightsProjection;
 }
```

In order to draw the shadow map, we need to draw the scene as seen by the headlights, we need to create a corresponding ViewProjection matrix. This is almost identical to setting up these matrices for the camera. 2 notes:

The aspect ratio of the projection matrix only needs to equal the apsect ratio of your screen in case you are setting up your camera. In this case, I defined 1f as the aspect ratio, so our light will see a square.
The distance of the near and far clipping planes will correspond to black and white in our distance map.

We need to pass this matrix to our graphics card before drawing our trianglestrip, and before drawing our models. So add this line to our Draw method:

```csharp
 effect.Parameters["xLightsWorldViewProjection"].SetValue(Matrix.Identity * lightsViewProjectionMatrix);
```

Note that the ViewProjection matrix is multiplied with the World matrix (the Identity matrix in the case of the street) to obtain the WorldViewProjection matrix.

Add this line to the DrawModel method:

```csharp
 currenteffect.Parameters["xLightsWorldViewProjection"].SetValue(worldMatrix * lightsViewProjectionMatrix);
```

Now we still need to select the proper technique to render our scene. As we’ll need to render our scene a second time with a later technique later on, now would be a nice time to put the whole contents of our Draw method in another method, DrawScene:

```csharp
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
```

This method will render our street, and 2 cars using the technique specified in the argument. Call this method from your Draw method like this:

```csharp
 protected override void Draw(GameTime gameTime)
 {
     device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);

     DrawScene("ShadowMap");

     base.Draw(gameTime);
 }
```

Now when you run this code, you should see the image below. It might look pretty strange at first sight, but it’s perfect: it is the depth map of the scene as seen by the headlights of the car. The more white the pixel, the farther away it is from the headlights.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-12Shadow2.jpg?raw=true)

## Our HLSL file

```csharp
float4x4 xWorldViewProjection;
float4x4 xLightsWorldViewProjection;
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
```

## The XNA code

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
             ambientPower = 0.2f;
 
             lightPos = new Vector3(-18, 5, -2);
             lightPower = 1.0f;
 
             Matrix lightsView = Matrix.CreateLookAt(lightPos, new Vector3(-2, 3, -10), new Vector3(0, 1, 0));
             Matrix lightsProjection = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver2, 1f, 5f, 100f);
 
             lightsViewProjectionMatrix = lightsView * lightsProjection;
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
 
             DrawScene("ShadowMap");
 
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
                 }
                 mesh.Draw();
             }
         }
 
     }
 }
```

## Next Steps

[Rendering our scene into a texture](Riemers3DXNA3hlsl13rendertotexture)
